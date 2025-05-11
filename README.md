### Smart-Wallet-Tracking ###
The goal of this software is to make surface level blockchain analysis accessible in terms of hardware requirements and ease of validation. It is optimized for low-resource machines and builds a bare bones database containing simplified transaction data for faster 
analysis about groups of bitcoins likely owned by the same user. The tool treats equivalent Bitcoin addresses as belonging to the same source, even if they use different formats (e.g., legacy, SegWit, or Taproot). Its hyperfocus, although limiting, allows for 
transaction information to be condensed to increase speed and allow for accessibility - a common spend clustering graph of all blockchain transacitons up to block 800,000 is only 17 GB. The library supports parallel processing to speed up analysis on large datasets.
If you’re working with a powerful machine or a computing cluster, you can enable multi-core processing using Ray with minimal setup. Traditional libraries are optimized for people who have computational resources, are highly generalizable (extra work to tailor), and often fail to treat different address formats from the same key as equivalent.  
### Dependencies ###
- psycopg2
- ray
- re
- hashlib
- base58
- coincurve
- time
- os
- Bitcoin Core v29
- PostgreSQL v17
## What things do ##
- create_tables.sql: Creates the table and schema for the database
- derivedUndefinedAddresses: Derived bitcoin addresses from public keys
- extract_bitcoin_data_beta.py: Extract detailed transaction data from a Bitcoin node via its RPC (Remote Procedure Call) and save the data as a CSV file
- populate_database.py: Automates several Bitcoin blockchain data processes such as deriving missing addresses from public keys, recovering revealed public keys from transaction inputs, normalizing address hashes, and clustering addresses using the common spend heuristic to identify entities controlling multiple addresses.
### Using the Library ###
1.) Download the blockchain with a Bitcoin Core full node and populate a postgreSQL database with that data using extract_bitcoin_data_beta.py. Please follow the steps in the extract_bitcoin_data_beta README to do so. This portion requires ~700GB of storage for a full node, but that can be red
Run it in the command prompt with:
```
py extract_bitcoin_data_beta.py
```
Or
```
python extract_bitcoin_beta.py
```
You will be prompted for a starting height. This is the height the code will begin parsing transactions from, note it down so you can stop it and restart it later if needed.
Note: This step will take a while. To avoid corrupting your Bitcoin node, only use the “bitcoin-cli stop” command in the command prompt at the daemon file path and allow full shutdown before closing. You can use task manager for this purpose as well.
2.) Enter the details of your postgresql server, then run the following script. This will take ~2 days to run on non performant systems - but faster drive speeds (such as NVME SSDs/RAID arrays with good partitioning) will lower that significantly. Change the memory settings as necessary to fit your system, they are by default set for a system with 20 GB of ram free.: 
```
populate_database.py
```
This script parses and subsequently builds a table of normalized hashes. Finally, it trims the now redundant parts of the database if desired. Normalizing hashes is important because a hash can produce many sub hashes. For example, a single public key can produce a traditional address, segwit address, etc.. Furthermore, this function normalizes the database for multisig addresses through the multi_output_hashes table. Options for assuming shared multisig ownership or non shared are available. 
4.) Create an edge set for common spend clustering, and load into memory to find the weakly connected components. 
```
commonSpendCluster(db_path = "YOUR_PATH_HERE")
```
OR
```
neo4jcommonSpendCluster(db_path = "YOUR_PATH_HERE")
```
5.) Calculate cluster balances through time
### Definitions:
Normalized Hash:
A Bitcoin public key can be hashed in several ways. For example, pubkey1 -> legacyaddr1 & pubkey1 -> otheraddr1. Further, some transactions contain several public keys for a single output, such as multisigs. In linking transactions via heuristic clustering, it is first necessary to link public keys to all of their child nodes via a unique identifier. This is also known as a normalized hash.
Heuristic Cluster:
An educated guess as to which wallets belong to a specific entity. For example, common spend clustering assumes all input wallets in any transaction belong to the same owner, as they must control all the private keys to transact with these wallets simultaneously. 
Transaction:
The locking and unlocking of an unspent transaction output (UTXO) via a script. Most commonly, hashes of the private key are used. 

