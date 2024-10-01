# Decentralized-cybersecurity-via-blockchain
college 2nd year group project
from web3 import Web3
import json
import time

# Connect to an Ethereum Node (Local, Infura, Ganache)
infura_url = "https://mainnet.infura.io/v3/YOUR_INFURA_PROJECT_ID"
web3 = Web3(Web3.HTTPProvider(infura_url))

# Check if connected to blockchain
if not web3.isConnected():
    print("Failed to connect to Ethereum")
    exit()

# Contract Address (replace with actual contract address)
contract_address = web3.toChecksumAddress('0xYourContractAddress')

# ABI (Application Binary Interface) of the smart contract (replace with your ABI)
contract_abi = json.loads('[...]')  # Paste your contract ABI here

# Load the smart contract
contract = web3.eth.contract(address=contract_address, abi=contract_abi)

# Define your wallet address and private key
wallet_address = web3.toChecksumAddress("0xYourWalletAddress")
private_key = "YourPrivateKey"

def report_threat(description):
    # Create the transaction to report a threat
    nonce = web3.eth.getTransactionCount(wallet_address)
    
    txn = contract.functions.reportThreat(description).buildTransaction({
        'chainId': 1,  # Mainnet (use 1337 for local testing)
        'gas': 2000000,
        'gasPrice': web3.toWei('50', 'gwei'),
        'nonce': nonce
    })
    
    # Sign the transaction with your private key
    signed_txn = web3.eth.account.signTransaction(txn, private_key=private_key)
    
    # Send the transaction to the blockchain
    tx_hash = web3.eth.sendRawTransaction(signed_txn.rawTransaction)
    print(f"Threat reported with transaction hash: {web3.toHex(tx_hash)}")

def verify_threat(threat_id):
    # Create the transaction to verify a threat (only for contract owner)
    nonce = web3.eth.getTransactionCount(wallet_address)
    
    txn = contract.functions.verifyThreat(threat_id).buildTransaction({
        'chainId': 1,
        'gas': 2000000,
        'gasPrice': web3.toWei('50', 'gwei'),
        'nonce': nonce
    })
    
    # Sign the transaction
    signed_txn = web3.eth.account.signTransaction(txn, private_key=private_key)
    
    # Send the transaction to the blockchain
    tx_hash = web3.eth.sendRawTransaction(signed_txn.rawTransaction)
    print(f"Threat verified with transaction hash: {web3.toHex(tx_hash)}")

def get_threat(threat_id):
    # Fetch threat details from the blockchain
    threat = contract.functions.getThreat(threat_id).call()
    threat_details = {
        'id': threat[0],
        'description': threat[1],
        'reportedBy': threat[2],
        'timestamp': time.strftime('%Y-%m-%d %H:%M:%S', time.gmtime(threat[3]))
    }
    return threat_details

def is_threat_verified(threat_id):
    # Check if a threat has been verified
    verified = contract.functions.isThreatVerified(threat_id).call()
    return verified

# Example usage
if __name__ == "__main__":
    # Report a new threat
    report_threat("Malicious IP detected attempting SSH brute force attack.")
    
    # Check if a threat is verified
    threat_id = 1
    print(f"Threat {threat_id} verified: {is_threat_verified(threat_id)}")
    
    # Get threat details
    threat_info = get_threat(threat_id)
    print(f"Threat details: {threat_info}")
    
    # Verify the threat (only contract owner)
    verify_threat(threat_id)
