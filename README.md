# team5-vehicle-mgmt

Channel with 4 Organizations and 1 peer in each organization

4 organizations are MFR,DLR,RTO,SCR
MFR is the orderer in this channel

########Get the code from github
sudo git clone -b master https://github.com/TSBlockchain/team5-vehicle-mgmt.git

After the above command a new folder will be created called "team5-vehicle-mgmt"

Change/Switch directory to 
cd team5-vehicle-mgmt

check the folder for this file ".env" (this is hidden file to see the hidden files in folder Ctrl+H) if this file is not there copy this from other project.

#### Clean up config folders first 
sudo rm -R crypto-config/*

####### set the fabric path (no need to change anything just run this command it will set the path automatically)
export FABRIC_CFG_PATH=$PWD

sudo rm config/*

sudo ../bin/cryptogen generate --config=./crypto-config.yaml

sudo ../bin/configtxgen -profile OneOrgOrdererGenesis -outputBlock ./config/genesis.block 

sudo ../bin/configtxgen -profile OneOrgChannel -outputCreateChannelTx ./config/channel.tx -channelID mychannel

update environment variables in docker-compose.yml

### command to stop docker
sudo docker stop $(sudo docker ps -aq)

### command to remove docker
sudo docker rm $(sudo docker ps -aq)

### command to down docker
sudo docker-compose -f docker-compose.yml down -d 

### command to start docker
sudo docker-compose -f docker-compose.yml up -d

####Create the channel genesis block (mychannel.block) using our channel configuration file channel.tx 

sudo docker exec -e "CORE_PEER_LOCALMSPID=MfrMSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/mfr.vm.com/users/Admin@mfr.vm.com/msp" -e "CORE_PEER_ADDRESS=peer0.mfr.vm.com:7051" cli peer channel create -o orderer.vm.com:7050 -c mychannel -f /etc/hyperledger/configtx/channel.tx

#### Peer needs to join the channel using channel genesis block mychannel.block
### Here we have 4 peers 1 in each organization
 
sudo docker exec -e "CORE_PEER_LOCALMSPID=MfrMSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/mfr.vm.com/users/Admin@mfr.vm.com/msp" -e "CORE_PEER_ADDRESS=peer0.mfr.vm.com:7051" cli peer channel join -b mychannel.block

sudo docker exec -e "CORE_PEER_LOCALMSPID=DlrMSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/dlr.vm.com/users/Admin@dlr.vm.com/msp" -e "CORE_PEER_ADDRESS=peer0.dlr.vm.com:7051" cli peer channel join -b mychannel.block

sudo docker exec -e "CORE_PEER_LOCALMSPID=RtoMSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/rto.vm.com/users/Admin@rto.vm.com/msp" -e "CORE_PEER_ADDRESS=peer0.rto.vm.com:7051" cli peer channel join -b mychannel.block

sudo docker exec -e "CORE_PEER_LOCALMSPID=ScrMSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/scr.vm.com/users/Admin@scr.vm.com/msp" -e "CORE_PEER_ADDRESS=peer0.scr.vm.com:7051" cli peer channel join -b mychannel.block


### In fabric-samples/chaincode/fabcar/go folder fabcar.go 
### We will run fabcar.go now in the following steps

##### Install (copy) chaincode to peer0 of Mfr
sudo docker exec -e "CORE_PEER_LOCALMSPID=MfrMSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/mfr.vm.com/users/Admin@mfr.vm.com/msp" -e "CORE_PEER_ADDRESS=peer0.mfr.vm.com:7051" cli peer chaincode install -n fabcar -v 1.0 -p github.com/fabcar/go -l golang

##### Instantiate the chaincode
sudo docker exec -e "CORE_PEER_LOCALMSPID=MfrMSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/mfr.vm.com/users/Admin@mfr.vm.com/msp" -e "CORE_PEER_ADDRESS=peer0.mfr.vm.com:7051" cli peer chaincode instantiate -o orderer.vm.com:7050 -C mychannel -n fabcar -l golang -v 1.0 -c '{"Args":[""]}' -P "AND ('MfrMSP.member')"

##### Invoke/Execute initLedger operation of fabcar chaincode
sudo docker exec -e "CORE_PEER_LOCALMSPID=MfrMSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/mfr.vm.com/users/Admin@mfr.vm.com/msp" -e "CORE_PEER_ADDRESS=peer0.mfr.vm.com:7051" cli peer chaincode invoke -o orderer.vm.com:7050 -C mychannel -n fabcar -c '{"function":"initLedger","Args":[""]}'

##### Invoke/Execute createCar operation of fabcar chaincode
sudo docker exec -e "CORE_PEER_LOCALMSPID=MfrMSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/mfr.vm.com/users/Admin@mfr.vm.com/msp" -e "CORE_PEER_ADDRESS=peer0.mfr.vm.com:7051" cli peer chaincode invoke -o orderer.vm.com:7050 -C mychannel -n fabcar -c '{"function":"createCar","Args":["CAR11", "Maruti","Swift", "Red", "Purshotam"]}'

##### Invoke/Execute queryAllCars operation of fabcar chaincode
sudo docker exec -e "CORE_PEER_LOCALMSPID=MfrMSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/mfr.vm.com/users/Admin@mfr.vm.com/msp" -e "CORE_PEER_ADDRESS=peer0.mfr.vm.com:7051" cli peer chaincode invoke -o orderer.vm.com:7050 -C mychannel -n fabcar -c '{"function":"queryAllCars","Args":[""]}'


  
