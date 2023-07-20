# Vote_demo23
Block-chain Voting System Project Using Python

* Install required libraries: bash

pip install Flask requests

* 1.)Import necessary libraries:

from hashlib import sha256
import json
import time
from flask import Flask, request

app = Flask(__name__)


* 1.)Create basic blockchain structures:


class Block:
    def __init__(self, index, previous_hash, timestamp, data, hash):
        self.index = index
        self.previous_hash = previous_hash
        self.timestamp = timestamp
        self.data = data
        self.hash = hash


def calculate_hash(index, previous_hash, timestamp, data):
    return sha256(f"{index}{previous_hash}{timestamp}{data}".encode("utf-8")).hexdigest()


def create_genesis_block():
    # Manually create the first block (genesis block)
    return Block(0, "0", time.time(), "Genesis Block", calculate_hash(0, "0", time.time(), "Genesis Block"))


# Initialize the blockchain with the genesis block
blockchain = [create_genesis_block()]


* 1.)Implement the voting contract:

class VotingContract:
    def __init__(self):
        self.candidates = {}
        self.voters = set()

    def add_candidate(self, candidate_name):
        if candidate_name not in self.candidates:
            self.candidates[candidate_name] = 0

    def vote(self, candidate_name, voter_id):
        if candidate_name in self.candidates and voter_id not in self.voters:
            self.candidates[candidate_name] += 1
            self.voters.add(voter_id)
            return True
        return False


# Initialize the voting contract
voting_contract = VotingContract()


* 1.)Implement the blockchain API:


@app.route("/new_transaction", methods=["POST"])
def new_transaction():
    data = request.get_json()
    required_fields = ["candidate", "voter_id"]
    if not all(field in data for field in required_fields):
        return "Invalid transaction data", 400

    candidate = data["candidate"]
    voter_id = data["voter_id"]

    # Process the transaction and add it to the blockchain
    index = len(blockchain)
    previous_hash = blockchain[-1].hash
    timestamp = time.time()
    transaction_hash = calculate_hash(index, previous_hash, timestamp, f"{candidate}{voter_id}")

    # Simulate the contract execution
    if voting_contract.vote(candidate, voter_id):
        new_block = Block(index, previous_hash, timestamp, data, transaction_hash)
        blockchain.append(new_block)
        return "Transaction added successfully", 201
    else:
        return "Transaction failed. Candidate may not exist or voter has already voted.", 400


@app.route("/chain", methods=["GET"])
def get_chain():
    chain_data = []
    for block in blockchain:
        chain_data.append(
            {
                "index": block.index,
                "timestamp": block.timestamp,
                "data": block.data,
                "hash": block.hash,
            }
        )
    return json.dumps(chain_data)


if __name__ == "__main__":
    app.run(debug=True)
    

* 1.)Run the Flask app:
Save the code in a file named blockchain_voting_system.py and run it:

python blockchain_voting_system.py

The app will start running on http://127.0.0.1:5000/.

