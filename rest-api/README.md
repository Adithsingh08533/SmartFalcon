Hyperledger Fabric REST API – Account Management System
A RESTful API built on Hyperledger Fabric for managing account operations (create, read, update, history tracking) using a custom chaincode implementation.

📋 Table of Contents

Overview
Prerequisites
Installation & Setup
API Endpoints
Testing
Troubleshooting
Architecture
License


🎯 Overview
This project demonstrates a complete Hyperledger Fabric application with:

Custom Go chaincode (accountcc) for account management
REST API server for easy client interaction
Docker-based deployment
Full transaction history tracking on the blockchain

Key Features:

Create and manage accounts with dealer ID, MSISDN, balance tracking
Update account balances and metadata
Query complete transaction history
Secure blockchain-backed data persistence


🚀 Prerequisites
System Requirements:

Windows 10/11 with WSL 2 (Ubuntu 20.04 or later)
Docker Desktop with WSL integration
Git
Go 1.21 or higher
8GB+ RAM
20GB+ disk space

Software Installation:

Docker Desktop must have WSL integration enabled
jq for JSON processing


⚙️ Installation & Setup
1. Enable Docker in WSL
Open Docker Desktop → Settings → Resources → WSL Integration
Enable integration with your Ubuntu distribution → Apply & Restart
Verify Docker is accessible:
bashdocker --version
docker-compose --version
2. Clone Fabric Samples Repository
bashcd ~
git clone https://github.com/hyperledger/fabric-samples.git
cd fabric-samples
3. Install Fabric Components
bash./install-fabric.sh docker binary
This downloads:

Hyperledger Fabric Docker images
Fabric binaries (peer, orderer, etc.)

4. Start the Test Network
bashcd ~/fabric-samples/test-network
./network.sh up createChannel -c mychannel -ca
If you encounter a jq error:
bashsudo apt update
sudo apt install jq -y
Set anchor peers:
bash./network.sh setAnchorPeer -c mychannel -o 1
./network.sh setAnchorPeer -c mychannel -o 2
5. Deploy the Account Chaincode
bashcd ~/fabric-samples/test-network
./network.sh deployCC -c mychannel -ccn accountcc -ccp ../asset-transfer-work/chaincode-go -ccl go
6. Run the REST API
Navigate to the REST API directory:
bashcd ~/fabric-samples/asset-transfer-work/rest-api
Option 1: Run Directly with Go (Recommended for Development)
Set environment variables:
bashexport CHANNEL_NAME=mychannel
export CHAINCODE_NAME=accountcc
export MSP_ID=Org1MSP
export CERT_PATH=$HOME/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/signcerts/cert.pem
export KEY_PATH=$HOME/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/keystore/
export TLS_CERT_PATH=$HOME/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export PEER_ENDPOINT=localhost:7051
export GATEWAY_PEER=peer0.org1.example.com
Run the API:
bashgo run main.go
Option 2: Run with Docker
bashdocker build --network=host --no-cache -t asset-rest:latest .
docker run --network=host \
  -e CHANNEL_NAME=mychannel \
  -e CHAINCODE_NAME=accountcc \
  asset-rest:latest
The REST API will be available at http://localhost:8080

📌 API Endpoints
Create Account
bashcurl -X POST http://localhost:8080/accounts \
-H "Content-Type: application/json" \
-d '{
  "key": "acc001",
  "DEALERID": "D001",
  "MSISDN": "9876543210",
  "MPIN": "1234",
  "BALANCE": "10000",
  "STATUS": "ACTIVE",
  "TRANSAMOUNT": "0",
  "TRANSTYPE": "INIT",
  "REMARKS": "Initial account"
}'
Response:
json{"result":"created","key":"acc001"}
Read Account
bashcurl http://localhost:8080/accounts/acc001
Response:
json{
  "DEALERID": "D001",
  "MSISDN": "9876543210",
  "MPIN": "1234",
  "BALANCE": 10000,
  "STATUS": "ACTIVE",
  "TRANSAMOUNT": 0,
  "TRANSTYPE": "INIT",
  "REMARKS": "Initial account"
}
Update Account
bashcurl -X PUT http://localhost:8080/accounts/acc001 \
-H "Content-Type: application/json" \
-d '{
  "BALANCE": "15000",
  "REMARKS": "Balance updated"
}'
Response:
json{"result":"updated","key":"acc001"}
Get Account History
bashcurl http://localhost:8080/accounts/acc001/history
Response:
json[
  {
    "IsDelete": false,
    "Timestamp": {"seconds": 1759397987, "nanos": 869326844},
    "TxId": "5c021a04...",
    "Value": {
      "BALANCE": 15000,
      "DEALERID": "D001",
      "MPIN": "1234",
      "MSISDN": "9876543210",
      "REMARKS": "Balance updated",
      "STATUS": "ACTIVE",
      "TRANSAMOUNT": 0,
      "TRANSTYPE": "INIT"
    }
  }
]

🧪 Testing
Verify your network is running:
bashdocker ps
Expected containers:

peer0.org1.example.com
peer0.org2.example.com
orderer.example.com
ca_org1, ca_org2, ca_orderer

Test the complete workflow:
bash# 1. Create an account
curl -X POST http://localhost:8080/accounts -H "Content-Type: application/json" \
-d '{"key":"acc001","DEALERID":"D001","MSISDN":"9876543210","MPIN":"1234","BALANCE":"10000","STATUS":"ACTIVE","TRANSAMOUNT":"0","TRANSTYPE":"INIT","REMARKS":"Initial account"}'

# 2. Read the account
curl http://localhost:8080/accounts/acc001

# 3. Update the balance
curl -X PUT http://localhost:8080/accounts/acc001 -H "Content-Type: application/json" \
-d '{"BALANCE":"15000","REMARKS":"Balance updated"}'

# 4. Check transaction history
curl http://localhost:8080/accounts/acc001/history

🛠 Troubleshooting
Docker not accessible in WSL

Ensure Docker Desktop WSL integration is enabled
Restart Docker Desktop after enabling WSL integration

jq: command not found
bashsudo apt update && sudo apt install jq -y
Docker build timeout issues

Disable IPv6 in Docker Desktop settings
Try building with --network=host flag
Or run the API directly with Go instead of Docker

Chaincode deployment fails

Ensure Go 1.21+ is installed: go version
Verify the test network is running: docker ps

Cannot connect to peer

Check firewall settings
Verify environment variables are set correctly
Ensure certificates exist in the specified paths

Network cleanup (start fresh)
bashcd ~/fabric-samples/test-network
./network.sh down
./network.sh up createChannel -c mychannel -ca

🏗 Architecture
┌─────────────────┐
│   REST API      │
│  (Port 8080)    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Fabric Gateway  │
│     API         │
└────────┬────────┘
         │
         ▼
┌─────────────────┐     ┌─────────────────┐
│  Peer Org1      │────▶│  Peer Org2      │
│  (Port 7051)    │     │  (Port 9051)    │
└────────┬────────┘     └────────┬────────┘
         │                       │
         └───────────┬───────────┘
                     ▼
            ┌─────────────────┐
            │    Orderer      │
            │  (Port 7050)    │
            └─────────────────┘
Components:

Chaincode: Go-based smart contract for account management
REST API: Go HTTP server using Fabric Gateway SDK
Peers: Two organizations with one peer each
Orderer: Solo orderer for transaction ordering
CAs: Certificate authorities for each organization


📜 License
Apache-2.0 License (same as Hyperledger Fabric)

🔗 Reference Links

Hyperledger Fabric Documentation
Fabric Samples Repository


Author: Adith Singh
Project: Hyperledger Fabric Account Management System
Last Updated: October 2025