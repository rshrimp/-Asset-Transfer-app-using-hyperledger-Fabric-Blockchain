# Asset Transfer using hyperledger Fabric

This is a simple demonstration of setting up 2 orgs with 2 peers each. Using Solo orderer and Fabric CA for certificates.
It includes a node.js app for invoking chaincode.

## Transfer Consortium Setup
1. Two peers for each organization
2. Pre-created Member Service Providers (MSP) for authentication and identification
3. An Orderer using SOLO
4. _Testchannel_ – this channel is public blockchain both orgs have read and write access to it.

## Directory Structure
```
├── app				        
      ├── app.js
├── chaincode				        
      ├── transfer			
        ├── transfer.go
├── channels
├── crypto-config		
├── cli.yaml
├── configtx.yaml
├── crypto-config.yaml
├── crypto.sh to generate CA certs
├── docker-compose-transfer.yaml                  
├── peer.yaml
├── README.md
├── scripts
    ├──chaincodeInstallInstantiate.sh
    ├──cleanup.sh
    ├──createArtifacts.sh
    ├──creatChannels.sh
    ├──createLedgerEntries.sh
    ├──createTransferRequest.sh    
    ├──query.sh
    ├──queryAll.sh
    ├──setupNetwork.sh    
    ├──start_network.sh
    ├──stop_network.sh
```



* `cd transferCA`  

## Setup network
* Run the following command to kill any stale or active containers:

  `./scripts/cleanup.sh`

* Create artifacts ( genesis block and channel info) if need to be (orderer genesis block and channels)
_This script is not going to create crypto-config folder for certs as it is already generated.
But if you wish you can delete the crypto-config folder that was created before and then run crypto.sh script to generate the CA certs_

  `./scripts/createArtifacts.sh`


* start network with start option
  `./scripts/start_network.sh`

* `./scripts/setupNetwork.sh` this script creates channels, join channels, instantiates and installs chaincode, populates ledger with initial entries. It will also dump the entire ledgers at the end.

* `docker exec -it client.Org1 bash`

+ And register a user:

`fabric-ca-client register --id.name admin2 --id.type user --id.affiliation org1 --id.secret adminpw`

+ Finally enroll:

fabric-ca-client enroll -u http://admin2:adminpw@ca.Org1:7054 -M Admin2@Org1.com/msp


##### Now run asset transfer request.  

OPTION -1:

The script runs transfer request multiple times on the same asset id=123 to show the updates which are printed at the end of the script runs
`./scripts/createTransferRequest.sh`

```{
  "Snumber": "123",
  "Description": "5 High Strret, CA 75000 ",
  "Owner": "Rishi Shimpi",
  "Status": "transferred",
  "TransactionHistory": {
    "createAsset": "Wed, 14 Mar 2018 19:04:51 UTC",
    "transferAssetWed, 14 Mar 2018 19:06:15 UTC": "Asset transferred from: John Doe to new owner: x  on:Wed, 14 Mar 2018 19:06:15 UTC",
    "transferAssetWed, 14 Mar 2018 19:07:48 UTC": "Asset transferred from: bb to new owner: y on:Wed, 14 Mar 2018 19:07:48 UTC",
    "transferAssetWed, 14 Mar 2018 19:07:53 UTC": "Asset transferred from: cc to new owner: z on:Wed, 14 Mar 2018 19:07:53 UTC",
    "transferAssetWed, 14 Mar 2018 19:07:58 UTC": "Asset transferred from: dd to new owner: yy on:Wed, 14 Mar 2018 19:07:58 UTC"
  }
}```

OPTION -2: Running from the app.js
cd /app

node app.js
curl -s -X POST  http://localhost:4000/invoke -H "content-type: application/json" -d '{"Args":["transferAsset", "123", "Raju"]}'
OR open http://localhost:4000
