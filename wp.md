# [Ziesha Network](https://twitter.com/ZieshaNetwork) - Whitepaper

## Abstract

Ziesha is a new layer-1 cryptocurrency which uses Zero-Knowledge proofs as the back-end of its smart-contracts, focusing on a more scalable blockchain by compressing transactions through zkRollup-like circuits. In order to keep the protocol simple, the smart-contracts are expressed as mathematical constraints instead of bytecodes of a virtual machine. Ziesha incorporates a special, built-in smart-contract in its genesis block, called the Main Payment Network (MPN), which is a simple payment circuit, allowing Ziesha to verify correct execution of hundreds of transfers inside a single Groth16 proof. Validators are required to execute this contract as a part of their block creation process.

## 1. Introduction

Bitcoin first appeared in 2009, introducing itself as a digital currency that can be transferred on a peer-to-peer network. The technology behind it, known as *blockchain*, became massively popular when people figured out that it can be used for more than just money transfer. The side-effect of this discovery was that blockchains couldn't handle the huge number of transactions coming from the users anymore, which caused skyrockets in transaction fees, making the currencies almost unusable at peak times.

### 1.1 Towards lighter blockchains

In a basic model, if we define a blockchain as a chain of blocks each contatining a limited number of transactions, we can approximate the number of transactions it can handle per second, using the following equation:

$$Throughput=\frac{BlockSize}{TransactionSize.BlockTime}$$

$BlockSize$ determines how many transactions can reside in a block, $BlockTime$ determines how often a block is generated and $TransactionSize$ is the average size of a transaction in the target blockchain. In a quick glance, we can conclude that $Throughput$ can be increased by:

 1. Increasing $BlockSize$ (More transactions per block, e.g Bitcoin Cash)
 2. Decreasing $BlockTime$ (Faster generation of blocks, e.g Solana)
 3. Decreasing $TransactionSize$ (E.g Layer-2 solutions: Payment Channels, Rollups, etc.)

We can also predict the size of the full blockchain using the following equation:

$$BlockchainSize=\frac{t.BlockSize}{BlockTime}$$

And if we take the derivative of $BlockchainSize$ with respect to time, we can find out the rate with which the blockchain size increases over time:

$$\frac{dBlockchainSize}{dt}=\frac{BlockSize}{BlockTime}$$

This clearly indicates that, having a big $BlockSize$ or small $BlockTime$, both result in a faster increase of $BlockchainSize$. It is claimed, both theoritically and practically, that blockchains that grow too quickly suffer from ***centralization***. This is due to the fact that it is harder for a new node to sync with the network when the blockchain has got too big. And also, it gets harder for already running nodes to maintain an ultra-large database. It can be seen that reducing the $TransactionSize$ doesn't have any effect on the rate $BlockchainSize$ grows, so it can be concluded that: ***Decreasing the size of transactions is the most sensible approach towards building a blockchain that is scalable and maintainable at the same time.***

### 1.2 Transaction Compression (I.e Layer-2 solutions)

Reducing the size of transactions might seem impossible in the first glance, but some cryptographic tricks and methods have been invented to reduce the ***effective-size*** of transactions. These methods are known as "Layer-2" solutions nowadays.
In a layer-2 solution, the user transactions are not stored on the main chain but their ***claims*** are. These claims can be validated by the main chain (Layer-1) through ***validity proofs*** or falsified and punished through ***fraud proofs***. The simplest layer-2 solution ever invented is a payment-channel, which is the building block of the Bitcoin's Lightning Network. In a payment channel, two users put some funds on a payment channel as a deposit, and after that, start creating **messages** signed by both of them, in which it is stated the amount each party owns in that update, along with a timestamp. These messages are not uploaded on the main chain, reducing the overall traffic happening on the blockchain. A party can bring out his funds from the channel by uploading a withdrawal request containing the latest state which is signed by both parties. The request is not processed immediately, but after a delay, allowing the other party/parties to upload a newer state in case the first party was cheating and claiming an older state. The main chain (Layer-1) can punish the cheater accordingly. This is a simple example of a fraud-proof.

More complicated versions of similar ideas include Ethereum Plasma, in which a single operator is responsible for processing users' transactions on a merkle-tree of accounts, and only uploading the merkle-root of the account tree on the main chain. Users can detect malicious activity of the operator and make a fraud-proof accordingly.

In the case of **validity proofs**, the one who publishes the new state, provides a convincing proof that the state transition is valid, instead of challenging others to provide a **fraud proof**. This removes the need for delays and interactive challenges, providing a much better user-experience.

Common drawbacks of L2 solutions that rely on ***fraud proofs***:

* Receivers of transactions may be required to be online.
* Typically, they require trusting a centralized third party. Users can then punish the centralized party in case he started cheating, thus guaranteeing the network's security. You must be online to prove that the third party is cheating.
* These methods are not native to the host blockchain. The layer-2s are deployed as smart-contracts and users need to send their funds to these contracts in order to take advantage of them. Entering/exiting those contracts is not free.
* Strange things can happen if the centralized party goes offline.

### 1.3 Roll-up systems

Rollups are a category of layer-2 scaling solutions which try to move the computation effort of user transactions off-chain. Rollup systems typically maintain a single state that contains the balance/state of all their members, and only upload a **commitment** to that state on the main chian. Normally, the commitment is the hash of the state.

The validity of these commitments is either approved by validity-proofs or rejected by fraud-proofs. As previously mentioned, the layer-2 solutions that are based on fraud-proofs, need to consider a delay for those who want to withdraw their funds from the rollup contract, so that other members are able to build fraud-proofs in case of malicious activity. This is a huge regress in user-experience and therefore is not as popular as solutions based on validity-proofs.

Validity-proofs on the other hand, help the layer-1 to verify the correctness of state-transitions at the same time the commitment is uploaded. Therefore a withdrawal delay, or complicated interactive fraud-proofs are not needed. Rollup systems that are based on validity-proofs normally rely on succinct Zero-Knowledge proofs.

### 1.4 Data-availability of SNARK states

In case of a validity-proof-base Rollup contract, the rollup producer could go offline and stop producing new blocks (Intentionally or unintentionally). The payment system will stop and users' funds will be locked in the contract. The solution is to change the rollup operator by running an election on the main-chain.
VM-based blockchains (such as Ethereum) cannot enforce data availability of the SNARK state because they only enforce data availability of the chain history, not the actual SNARK state.

### 1.5 Constant-sized blockchains

A constant-sized blockchain can be built by having a rollup circuit that is able to also verify the SNARK proof of its previous state recursively, through recursive SNARKs. The most famous example of such a system is the MINA protocol. These systems typically have a single mother-circuit that tries to support all kind of blockchain activity (Money transfer, smart-contracts, etc).

 1. A single circuit is obviously less parallelizable.
 2. They have complicated circuits, making the system more likely to have security issues.
 3. The system is not flexible, since it relies on a single circuit.
 4. A single circuit (Although giving space efficiency) might not be also computationally efficient.

Although MINA successfully provided a lightweight blockchain that can be stored in a machine as small as a smartphone, it does not provide enough throughput. Practically speaking, it has been stated that MINA has a throughput of 22 Transactions Per Second.


## 2. Ziesha Network - Bringing L2 cleverness inside L1

Even though validity-proof based rollup systems are great tools for compressing transactions, we should admit that there are serious user-experience flaws in them when used in a higher layer. We design a new cryptocurrency, Ziesha, which incorporate concepts previously used as privacy or Layer-2 solutions in other chains into the core of a new blockchain, aiming to create a more scalable network with better privacy.

### 2.1 Zero Contracts :passport_control:

A Zero Contract in Ziesha will be the equivalent of a Smart Contract. In the Ziesha blockchain, contracts are proposed not to be written for a specific virtual machine (such as EVM). The contracts are proposed to be written in R1CS (the building block of zkSNARK circuits). The programmer uploads the verification keys of his R1CS contract (which can consist of multiple circuits) to the blockchain, and anyone can invoke these circuits and move from one state to another with a single transaction (which could be just a compressed version of thousands of transactions).

***Transaction Executors*** are proposed to be one of Ziesha's main building blocks. These machines regularly execute Zero-Contracts which will normally compress users' transactions using zero-knowledge proofs. Transaction Executors are pretty much the same as zkRollups operators in the Ethereum blockchain. Ziesha will be optimized to support Transaction Executors as they are within the core of the Ziesha blockchain and not a different layer-2 method. Normal transactions will be discouraged in Ziesha, and users will be forced to join an Executor's payment network to transact money.

By requiring the Executors to reveal the full-state of the SNARK contracts after being updated, Ziesha aims to solve the data availability problems current layer-2 zk-rollup solutions encounter. Nodes are proposed not to accept chains that do not reveal the latest full-state of all their contracts. Obviously, the previous full-states are safely deleted after each update.

### 2.2 On-chain storage of SNARK states

Ziesha nodes work in a way that only accept forks that have revealed the full-state of all their contract, i.e. they check if the hash of the given full-states correctly result into the claimed state-hashes. A longer subchain in which the full-state of a contract is unknown is deemed worthless and not accepted by the network.

Ziesha's blockchain keeps track of state sizes, preventing them from getting too large. If the state-size is increased, executors must pay for extra bytes.
Since the only overhead of contract creation, and the update of contracts, is the size of the submitted transaction, Ziesha will not have such complexities. Fees will be based on the price of each byte submitted.

### 2.3 The Main Payment Network
The Main Payment Network (MPN) is a special, builtin smart-contract that is created in the genesis-block of the Ziesha Protocol. It manages a merkle-tree of millions of accounts that can transfer ℤ with each other with nearly zero-cost transactions. It uses the Groth16 proving system and has a fixed number of transaction slots per update. MPN inherits its consensus and data-availability guarantees from the main blockchain. Ziesha wallets will all use this network as their primary payment mechanism and will only submit regular transactions when they want to withdraw their funds from this contract and enter them into another contract.


<img width="400" src="https://user-images.githubusercontent.com/4275654/188954000-450b32ad-c5e8-4714-9664-3afa40400508.png" alt="Deposit/Withdraw/Rsend/Zsend">

#### Circuit specifications

MPN-state is the merkle-root of an arity-4 Merkle-tree with 4^15 leaves (Depth: 15) that uses Poseideon hash function over Bls12-381 scalar field and each leaf is also equal to Poseidon hash of 4 scalar elements representing a MPN-account:

 1. Nonce
 2. Balance-hash
 3. Pub-key X
 4. Pub-key Y

`Pub-key X` and `Pub-key Y` represent the JubJub elliptic curve point that is registered for that account. JubJub is an elliptic curve that is defined on Bls12-381 scalar-field and is suitable for building EdDSA digital-signatures on a Groth16 R1CS circuit.

Balance-hash as the merkle-root of an arity-4 Poseidon merkle-tree fo 4^4 (64) elements and each leaf is is determined by calculating Poseidon hash on 2 elements:
 
 1. Token-Id
 2. Balance

Zero Contracts can contain 3 kinds of functions in them:

 1. Deposit functions (When funds are entering the contract)
 2. Update functions (When internal changes are happening in the state)
 3. Withdraw functions (When funds are quitting the contract)

The deposit/withdraw circuits of the MPN-contract allows Ziesha users to enter/exit their funds to/from the MPN. They can handle up to 64 deposits/withdrawals.

The update circuit of the MPN contract allows MPN users to ***batch up to 256 of their transactions into a single proof***.

### 2.4 Removal of Gas fees

Ethereum introduced the concept of *Gas fee* because it is hard to predict how much computation a particular input to a contract function will require. Without gas-fees, attackers could cause infinite loops and freeze the Ethereum network.

On the other hand, Ziesha does not need to put limitations like this since the execution of contracts is done by the executors and Ziesha nodes would only need to verify proofs of correct execution.

As previously mentioned, Ziesha contracts are basically just verification keys of ZK-proof circuits, so their size is constant. No matter how complex the state transitions or circuit definitions are, the only overhead when creating a contract or submitting an update is a constant size transaction, thus there is no need for Ziesha nodes to track computation power used in order to execute a function inside a Zero-Contract, introducing a huge progress in user-experience.

### 2.5 Main-chain consensus

In the main chain, consensus is achieved through a Proof-of-Stake protocol, in which time is divided into epochs and slots and the slot winner is determined through a VRF. Since building a block requires computational effort, the risk of a Nothing-at-Stake attack is greatly reduced.

### 2.6 Executor election

To avoid catastrophic updates and wasteful computation-intensive Zero-Knowledge proofs, an election protocol should be designed in which (Statistically) only one Executor is allowed to produce proof per time slot. Ziesha tokens staked on a specific contract will be proportional to the chance of being elected.

### 2.7 Incentives

Providing consensus or executing contracts and proving their execution are the **two ways one can contribute to the Ziesha Network**. ***Validators*** contribute to the consensus, whereas **Executors** execute zkSNARK contracts and provide the proofs. Nodes do not have to execute the contracts, they will only check the validity of state transitions using the proofs provided by the Executors.

In Ziesha, validators are steadily rewarded in Ziesha tokens using PoS as their consensus algorithm. Validators have the option to become Executors and contribute to the network by executing contracts. Each contract has an Executor who is elected based on the funds staked on the contract. Fees collected from users' transactions reward the executors. Rewarding mechanisms are defined in contracts. In other words, if a contract has no defined reward mechanism, it is unlikely that an Executor will be willing to execute it.


## 4. Tokenomics

![](https://i.imgur.com/qjCjr4Q.png)


### 5.4 Roadmap

We have some goals that are not yet added to the Roadmap but are among our ideas. Join our community to discuss them!

![](https://i.imgur.com/tEqoRuD.png)

This roadmap is an estimate and subject to change.

## 4. Team


 * [Keyvan](https://github.com/keyvank)

   ![](https://i.imgur.com/LbGXFoA.jpg =205x250)


 * [Rues](https://github.com/ruesandora)

   ![](https://i.imgur.com/R3xx7io.jpg =200x250)

## Disclaimer

Opinions, ideas, and statements shared in this update are delivered with numerous assumptions, risks, and uncertainties which are subject to change over time. 

There are multiple risk factors, including those related to blockchain, cryptographic systems, and technologies generally, as well Ziesha's business, operations and results of operations, that could cause actual results or developments anticipated not to be realized or, even if substantially realized, to fail to achieve any or all of the benefits that could be expected therefrom. 

We reserve the right to unilaterally, completely, or partially change plans, expectations, and intentions stated herein at any time and for any reason, in our sole and absolute discretion, and we undertake no obligation to update publicly or revise any forward-looking statement, whether as a result of new information, future developments, or otherwise. 

ACCORDINGLY, WE RECOMMEND THAT YOU DO NOT RELY ON, AND DO NOT MAKE ANY FINANCIAL DECISION OR INVESTMENT BASED ON, THE STATEMENTS CONTAINED IN THIS UPDATE OR ANY OF OUR UPDATES/ARTICLES — INCLUDING BUT NOT LIMITED TO ANY SELLING OR TRADING OF ZIESHA TOKENS, ETHER, OR ANY OTHER CRYPTOGRAPHIC OR BLOCKCHAIN TOKEN, OR THE SECURITIES OF ANY COMPANY.
 
The views, opinions, and statements made in this update are those of an individual author and not those of any institution, University, or legal entity operating within the jurisdiction of any country.

There is no association between these views, opinions, and statements and any for-profit or non-profit entity, particularly with Universities, Foundations, and other Agencies located within any country.

Any perception of such an association is purely accidental, and will be rectified im
