This is a guide on how to set up a Theta Testnet on Ubuntu running on Windows 10. Here is a 10 minute video demo that shows these steps in action:

https://youtu.be/e7LvmtNhQOk

First you need to set up Windows Subsystem for Linux.  Here are instructions from Microsoft on how to do that:

https://docs.microsoft.com/en-us/windows/wsl/install-win10

At step 6 above, I installed the Ubuntu 20.04 LTS Distribution. Once the Ubuntu app is installed on your Windows 10 system, launch it and a terminal window will appear and it will initialize the Ubuntu environment and ask you for a user name and password.

Once the Ubuntu terminal is ready to accept commands, enter in the following to prepare for building the theta node and theta rpc server:
````
sudo apt-get update
sudo apt-get install build-essential gcc make git bzr jp
sudo wget https://golang.org/dl/go1.17.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.17.linux-amd64.tar.gz
````

Before we start building we need to add some environment variables to the .bashrc.  I also created aliases for starting up the Theta Node and Theta RPC Server once they are built.

Add the following lines to the end of your .bashrc:
````
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export PATH=$PATH:./:$GOROOT/bin:$GOPATH/bin
export THETA_HOME=$GOPATH/src/github.com/thetatoken/theta
export THETA_ETH_RPC_ADAPTOR_HOME=$GOPATH/src/github.com/thetatoken/theta-eth-rpc-adaptor
alias start-theta-node='cd $THETA_HOME && theta start --config=../privatenet/node_eth_rpc --password=qwertyuiop'
alias start-theta-rpc='cd $THETA_ETH_RPC_ADAPTOR_HOME && theta-eth-rpc-adaptor start --config=../privatenet/eth-rpc-adaptor'
````
Now execute your .bashrc:
````
. .bashrc
````
Create a shell script on Ubuntu called build-theta.sh and copy this into it:
````
#!/bin/bash
read -p "Caution: about to delete the go folder, hit ctrl-c if that is not OK or enter to continue "
set -x
set -e
cd $HOME
rm -rf .thetacli
sudo rm -rf go
mkdir go
cd go
#clone and build the theta node component
git clone https://github.com/thetatoken/theta-protocol-ledger.git $GOPATH/src/github.com/thetatoken/theta
cd $THETA_HOME
#this branch switch is crucial or it won't work
git checkout privatenet
export GO111MODULE=on
make install
cp -r ./integration/privatenet ../privatenet
mkdir ~/.thetacli
cp -r ./integration/privatenet/thetacli/* ~/.thetacli/
chmod 700 ~/.thetacli/keys/encrypted

#now clone and compile the rpc component
cd $GOPATH/src/github.com/thetatoken
rm -rf $THETA_ETH_RPC_ADAPTOR_HOME
git clone https://github.com/thetatoken/theta-eth-rpc-adaptor
#create the config.yaml for the rpc component
cd $THETA_ETH_RPC_ADAPTOR_HOME
mkdir -p ../privatenet/eth-rpc-adaptor
touch  ../privatenet/eth-rpc-adaptor/config.yaml
echo "theta:" >> ../privatenet/eth-rpc-adaptor/config.yaml
echo "  rpcEndpoint: \"http://127.0.0.1:16888/rpc\" " >> ../privatenet/eth-rpc-adaptor/config.yaml
echo "rpc:" >> ../privatenet/eth-rpc-adaptor/config.yaml
echo "  enabled: true" >> ../privatenet/eth-rpc-adaptor/config.yaml
echo "  httpAddress: \"127.0.0.1\" " >> ../privatenet/eth-rpc-adaptor/config.yaml
echo "  httpPort: 18888" >> ../privatenet/eth-rpc-adaptor/config.yaml
echo "  wsAddress: \"127.0.0.1\"" >> ../privatenet/eth-rpc-adaptor/config.yaml
echo "  wsPort: 18889" >> ../privatenet/eth-rpc-adaptor/config.yaml
echo "  timeoutSecs: 600 " >> ../privatenet/eth-rpc-adaptor/config.yaml
echo "  maxConnections: 2048" >> ../privatenet/eth-rpc-adaptor/config.yaml
echo "log:" >> ../privatenet/eth-rpc-adaptor/config.yaml
echo "  levels: \"*:debug\"" >> ../privatenet/eth-rpc-adaptor/config.yaml
export GO111MODULE=on
make install
````
Now it is time to build it all.  We need to mark the script as exectuable and then run it in the Ubuntu terminal. Please make sure it is the same script name you used above to create the file:
````
chmod +x build-theta.sh
build-theta.sh
````

Once the Theta Node and RPC Server are built, it is time to run them.  Enter the following in the Ubuntu terminal window:
````
start-theta-node
````
Launch another Ubuntu terminal and enter this:
````
start-theta-rpc
````

Hopefully, you now have both the Theta node and the RPC server up and running.  The Theta node window will show lots of activity since it is continuously created blocks and adding them to the blockchain.

Now, launch Google Chrome and install Metamask:

https://metamask.io/download

Once Metamask is installed, it needs to be configured to use the local theta testnet.  Bring up the Metamask settings and add a new network:
````
Network Name:     Local Theta Testnet
New RPC URL:      http://127.0.0.1:18888/rpc
Chain ID:         366
Currency Symbol:  TFUEL
````
The last task with Metamask is to import one of the Theta Testnet accounts so you have access to TFUEL.  After configuring and switching to the Local Theta Testnet in Metamask, import an account using this private key:
````
93a90ea508331dfdf27fb79757d4250b4e84954927ba0073cd67454ac432c737
````
You should have plenty of TFUEL now.

Once Metamask is installed and configured to use the Local Theta Testnet, you can navigate to Remix to try it out:

https://remix.ethereum.org/



