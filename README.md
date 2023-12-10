## [1.1. Intro to L1](https://www.youtube.com/watch?v=hYNG0Guezpk)
- symetric/asymetric encryption
- digital signatures
- hash functions
- merkle tree / merkle patricia tree / verkle tree
- blockchain timeline:
    - pre-1970s symetric cryptography
    - 1970s asymetric cryptography: Diffie Helman Key Exchange ([ot](https://github.com/archit-p/simplest-oblivious-transfer))
    - 1980s David Chaum - Blind Signatures / Digicash
    - 1990s Adam Back - HashCash / [Wei Dai - B-Money](http://www.weidai.com/bmoney.txt) / [Bit Gold - Nick Szabo](https://nakamotoinstitute.org/bit-gold/)
    - 2000s Peer to peer networks: Freenet / Gnutella / Bit Torrent
    - 2008 [Bitcoin](https://github.com/bitcoinbook/bitcoinbook/blob/develop/ch08.asciidoc)
- blockchain components:
    - a p2p network (based on "gossip" protocol)
    - messages (txs) representing state transitions
    - consensus rules (determines valid state transitions)
    - a state machine that processes txs according to consensus
    - a chain of cryptographically secured blocks
    - a consensus algorithm to enforce the consensus rules
    - a game-theory incentive scheme
    - one or more open source software impementations of the above (clients)
- blockchain architecture:
    - network: all participants are equal (p2p), no "special" nodes, no server, no centralised service, no hierarchy
    - nodes:
        - accept and transmit txs
        - keep a mempool
        - provide network discovery (via DNS seed nodes / locally stored list) and routing functions
        - drop connections to misbehaving nodes
        - accept blocks and update local ledger
        - connections are based on proximity in a hash table, not on geographical proximity
    - block data structure: header, timestamp, parent hash, nonce, txs
    - storage data structures: organised in Merkle trees / Merkle Patricia trees
    - alternative data structures: DAG (Sui/IOTA)
    - modular blockchains: [deep dive](https://volt.capital/blog/modular-blockchains) / [blockspace](https://www.paradigm.xyz/2021/03/ethereum-blockspace-who-gets-what-and-why)
        - execution: smart contracts business logic impl. Can be moved away from the evm (Starknet)
        - settlement/consensus: agreement on state transitions. Is securing the updates to the execution layer
        - [data availability](https://ethereum.org/en/developers/docs/data-availability/): where the data is stored and how to make sure it's available to the participants in the system
            - sampling (DAS): a way for the network to check that data is available. Each node (including non-staking ones) download small, randomly selected subset of data, gaining high confidence that all the data is available
            - committees (DACs): trusted parties that provide, or attest to, data availability. PoS DAC are more secure because they direclty incentivize honest behavior
- [ethereum consensus](https://ethereum.org/en/developers/docs/consensus-mechanisms/)
    - Technically, proof-of-work and proof-of-stake are not consensus protocols by themselves, but they are often referred to as such for simplicity. They are actually Sybil resistance mechanisms and block author selectors; they are a way to decide who is the author of the latest block. It is this Sybil resistance mechanism combined with a chain selection rule that makes up a true consensus mechanism.
    - block addition has two parts: 
        - block producer selection
        - block acceptance
    - pre-merge:
        - Beacon Chain wasn't processing Mainnet transactions. Was only reaching consensus on its own state (active validators and their account balances - [specs](https://github.com/ethereum/consensus-specs#phase-0)).
    - post-merge:
        - Eth 1.0 continue to handle execution (process blocks, maintain mempool, manage and sync state). PoW has been taken out
        - Beacon Chain continues to handle PoS consensus (track chain's head, gossip, attest blocks, receive validator rewards)
    - [execution clients](https://ethereum.org/en/developers/docs/nodes-and-clients/#execution-clients)
    - [consensus clients](https://ethereum.org/en/developers/docs/nodes-and-clients/#consensus-clients)
    - [client diversity](https://ethereum.org/en/developers/docs/nodes-and-clients/client-diversity/)
    - block finality: 
        - tendermint chains can halt if they don't get enough votes
        - nakamoto continue to add blocks and may not achieve finality
        - Ethereum achives finality by checkpointing (epochs every ~6min)

## [1.2. Why Scalability](https://www.youtube.com/watch?v=IIve3zX2xnw)
- Introduction:
    - regular users can run an eth node ([vitalik](https://vitalik.ca/general/2021/05/23/scaling.html))
    - Ethereum goal is to keep hardware requirements low ([scaling](https://ethereum.org/en/developers/docs/scaling/))
    - blockchain performance is [hard to measure](https://a16zcrypto.com/posts/article/why-blockchain-performance-is-hard-to-measure/) and TPS can have little real value

- Layer 1 solutions
    - eth validators form comittees and votes are aggregated by them
    - solana moved away from gossip protocol => only the leader (block producer) needs to receive txs so nodes send them to the leader not to the other nodes
    - eth txs are ordered and executed sequentially => simple design but leads to MEV / poor horizontal scaling
    - solana / aptos / sui allow parallel processing of  (non dependent) txs
    - sharding:
        - eth will have 64 shards
        - [introduction](https://medium.com/@icebearhww/ethereum-sharding-and-finality-65248951f649)
        - vitalik's [overview](https://vitalik.ca/general/2021/04/07/sharding.html) & [limits](https://vitalik.ca/general/2021/05/23/scaling.html)
    - rollups:
        - tx execution outside L1
        - data / proof of txs is on L1
        - rollup smart contract on L1 that can enforce correct tx execution on L2 by using the tx data from L1
        - L1 holds the funds; L2 holds state and performs execution
        - txs are commited to main chain in bundles (they are rolled-up)
        - are either optimistic or zk ([fraud proof vs. validity proof](https://www.alchemy.com/overviews/validity-proof-vs-fraud-proof#:~:text=What%20is%20a%20validity%20proof,information%20shared%20between%20the%20two.))
        - validity / zk => operator sumbmits a new state transition AND a cryptographic proof (a zk-SNARK) that the new state is valid transition from the old state. The state transitions is validated by a smart contract on L1. Usually the zero knowledge is ignored and input/data involved is public.
        - optimistic => assumes that the nodes are behaving honestly. State transition can be rolled back by providing a "fraud proof". After a certain period the state transition is considered final on L1.
    - state channels: 2 txs on L1 | eth locked in a multisig contract | either party can exit/close the channel ([lightning network](https://lightning.network))
    - plasma chains ([eth docs](https://ethereum.org/en/developers/docs/scaling/plasma) / [vitalik article](https://vitalik.eth.limo/general/2023/11/14/neoplasma.html)): utilize fraud proofs like optimistic rullups but have the mass exit problem.
    - Polygon [PoS](https://wiki.polygon.technology/docs/pos/) has a three-layer [architecture](https://wiki.polygon.technology/docs/pos/polygon-architecture/):
        - Ethereum layer — a set of contracts on the Ethereum mainnet.
        - [Heimdall](https://forum.polygon.technology/t/matic-system-overview-heimdall/8323) layer
        - [Bor](https://wiki.polygon.technology/docs/pos/design/bor/) layer
    - [Skale](https://skale.space)
    - [Loom](https://loomx.io)
- Ethereum Scalability directions:
    - The L2 scaling transition - [everyone moving to rollups](https://ethereum-magicians.org/t/a-rollup-centric-ethereum-roadmap/4698)
    - The wallet security transition - everyone moving to [smart contract wallets](https://vitalik.ca/general/2021/01/11/recovery.html)
    - The privacy transition - making sure privacy-preserving funds transfers are available

## [1.3. Intro to L2s / Maths and Cryptography](https://www.youtube.com/watch?v=DIXFlD-2PGU)

- Bridges vs. L2
    - less functional and more specialized
    - atomic token transfers
    - lock and mint mechanism
    - quite widespread ([wormhole](https://wormhole.com/))

- [Optimistic rollups](https://ethereum.org/en/developers/docs/scaling/optimistic-rollups/) in details:
    - bundle multiple off-chain txs in batches => reduces fees for end users
    - use compression techniques
    - if unchallenged within the challange period => it's deemed valid and accepted on L1
    - others can continue to build on unconfirmed rollup block **BUT** txs results will be reversed if based on an incorrectly executed tx published previously
    - process:
        - dev sends tx off-chain to aggregator
        - aggregator = anyone with a bond => multiple aggregators on the same chain
        - aggregator decides how fees are paid (acc abstraction / meta-tx)
        - instant guarantee to dev that tx will be included in the block (or else aggregator loses their bond)
        - aggregator locally applies the tx & computes the new state root
        - aggregator sumbits a L1 tx (which contains the L2 tx & state root) and pays for gas (which contains tx & state root)
        - if block is found invalid => invalidity proved with `verify_state_transition(prev_state, block, witness)`:
            - slashes the malicious aggregator & any aggregator who built on top of the invaid block
            - rewards the prover with a portion of the prover's bond
- [types](https://threadreaderapp.com/thread/1527052511844212738.html) of fraud proof systems:
    - level 1: admin can upgrade the system within the challenge period
    - level 2: admin can upgrade the system within the challenge period, but is **permissioned** to allow a few others to run the fault proof
    - level 3: admin can upgrade the system within the challenge period, but fault proof is **permissionless**
    - level 4: admin can NOT upgrade the system until users have a chance to withdraw their funds
- tx compression:
    - L1 tx takes ~110 bytes. L2 tx takes ~12 bytes (due to superior encoding + other compression tricks)
- data availability
    - to recreate the state, tx data is needed
    - where is data stored and how it's available to participants in the system
- [StarkNet](https://docs.starknet.io/documentation/):
    - ZK-Tollup mode (on-chain validity proofs)
    - state diff is sent as calldata to L1 => anyone can reconstruct the current state of StarkNet
    - to update the state on L1 it's enough to send a valid proof, without information on the txs or particular changes this update caused. => more info must be provided to allow other parties to locally track StarkNet's state
- [zkEVM](https://scroll.io/blog/zkEVM):
    - vm designed to emulate the EVM by recreating all existing EVM opcodes
    - acts as a state machine, processes state transitions of L2 txs
    - generates validity proofs that confirm the accuracy of the off-chain state change
    - two ways to build dApps on zk-Rollups:
        - app-specific circuit (ASIC):
            - very limited (R1CS only supports addition and multiplication). Circuit == program representation used in a zero knowledge proof 
            - unique circuits for each dapp => reduced overhead for dapps
            - needs high-level dev expertise in circuit design
        - universal EVM for smart contract execution
- [(Proto) Danksharding](https://notes.ethereum.org/@vbuterin/proto_danksharding_faq#Proto-Danksharding-FAQ):
    - implements most of the logic and scaffolding for full Danksharding, but not yet implementing any sharding
    - the main feature is the introduction of a **blob-carrying transaction**, which is like a regular tx but it carries an extra piece of data called **blob**
        - blobs are extremely large (byte string up to ~125kB)
        - blobs are committed with [KZG commitment](https://dankradfeist.de/ethereum/2020/06/16/kate-polynomial-commitments.html) scheme and are forward compatible with DAS
        - blobs are not accessible to the EVM and deleted after 1-3 months. The EVM can only view a commitment to the blob. Blobs are stored by the consensus layer (beacon chain)
        - blob is a vector of 4096 field elements, numbers within the range 0 <= x < 52435875175126190479447740508185965837690552500527637822603658699938581184513
    - currlently rollups post txs on L1 using CALLDATA (proccessed by all nodes and stored on-chain permanently)
    - once proto-danksharding is rolled out, execution layer, rollups and users need to do no further work to finish the transition to full sharding
    - the main practical difference between [EIP-4488](https://eips.ethereum.org/EIPS/eip-4488) and [EIP-4844](https://eips.ethereum.org/EIPS/eip-4844) is that EIP-4488 attempts to minimize the changes needed today, whereas proto-danksharding makes a larger number of changes today so that few changes are required in the future to upgrade to full sharding
    - leads to a maximum usage of ~1MB/block (~2.5TB/year)
    - consensus layer can implement separate logic to auto-delete blob data after some time (e.g 30 days) and not dependent of [EIP-4444](https://eips.ethereum.org/EIPS/eip-4444) implementation
    - in the long run  full sharding will add ~40 TB of historical data per year => some history expiry mechanism is mandatory
    - historical data will be stored by rollups, block explorers, hobbyinsts, indexing protocols etc, not by the consensus protocol
    - learn more about EIP-4844 on [Bankless](https://www.youtube.com/watch?v=N5p0TB77flM) or [Optimism](https://www.youtube.com/watch?v=KQ_kIlxg3QA)

## [1.4. Maths and Cryptography](https://www.youtube.com/watch?v=vi5I2KcMPxo)

- Fully Homomorfic Encryption (FHE):
    - a form of encryption that allows arbitrary computation on encrypted data
    - an additional evaluation capability for computing over encrypted data without access to the secret key
    - the result of such encryption remains encrypted
    - extension of either symmetric-key or public-key cryptography
    - algebra example: considering `f(n) = e^n` homomorphism means `f(m + n) = e^(m + n) = e^m * e^n = f(m) * f(n)`
    - private computation: [fhEVM](https://www.zama.ai/fhevm) from [Zama.ai](https://www.zama.ai/)
- VRF explanation from [Algorand](https://medium.com/algorand/algorand-releases-first-open-source-code-of-verifiable-random-function-93c2960abd61)
- Modular arithmetic [introduction](https://www.khanacademy.org/computing/computer-science/cryptography/modarithmetic/a/what-is-modular-arithmetic):
    - multiples of B end up in the same spot: `A % B = (A + K*B) % B` for any integer K
    - A is congurent to B modulo C: `A == B mod C` (A and B are in the same equivalence class)
- Group theory:
    - set of elements and a binary operation (e.g. *)
    - must have these properties:
        - closure: for all a, b in G => a*b is also in G
        - associativity: (a * b) * c = a * (b * c)
        - identity element: a * e = e * a = a
        - inverse element: for each a in G there is a b (denoted as -a or a^-1) such that a * b = b * a = e
    - sub group: a set of elements which has the group properties is a subgroup of the original group
    - finite group: can be cyclic and has a generator
- Fields:
    - set of integers and 2 operations called addition and multiplication
    - field axioms:
        - associativity
        - commutativity
        - additive & multiplicative identity
        - additive & multiplicative inverses
        - distributivity of multiplication over addition: a * (b + c) = (a * b) + (a * c)
    - [finite fields](https://asecuritysite.com/encryption/finite): a set of elements such as the set of integers mod p whare p is a prime
    - order of the field == the number of elements in the field's set. For a finite field it's either:
        - prime
        - power of prime (extension field)
    - every finite field has a generator which can generate the elements in the set be exponentiating the generator
        - Z(5) = {0,1,2,3,4} => it's cyclic and has two generators, 2 and 3:
            - 2^1 = 2, 2^2=4, 2^3=3, 2^4=1
            - 3^1 = 3, 3^2=4, 3^3=2, 3^4=1
- Group homomorphisms
    - A homomorphism is a map between two algebraic structures of the same type, tha tpreserves the operations of the structures: f(x*y) = f(x)*f(y)
- Equivalence classes
    - x mod 7 = 6 => x can be 6, 13, 20 and they form an equivalence class
    this give us the basis of a one way fn
- Fermat's little theorem is useful for finding a multiplicative inverse
    - a^-1 == a^p-2(modp)

- Polynomial introduction
    - TODO

## [2.1. Understanding and Analysing Layer 2 ](https://www.youtube.com/watch?v=rxZTgcdBiyU)

## [2.2. Agnostic Layer 2 Transaction Lifecycle](https://www.youtube.com/watch?v=_H0fc4vSxq4)

## [2.3. Arbitrum Workshop / Transaction Lifecycle cont](https://www.youtube.com/watch?v=DypJJMo4GoQ)

## [2.4. Rollups v ZK Rollups/Decentralized Sequencers](https://www.youtube.com/watch?v=GIyzralsT2I)

## [3.1. L3s / Hyperchains](https://www.youtube.com/watch?v=Cjse_ZSFIKE)

## [3.2. Privacy in Layer 2 / Aztec Workshop](https://www.youtube.com/watch?v=lAyW5JkyFfc)

## [3.3. What are ZK EVMs Part 1 (Overview)](https://www.youtube.com/watch?v=2oJSoVqpH7k)

- zkVM:
    - [VMs](https://www.cs.nmt.edu/~doshin/t/s10/cs589/pub/10.SmithComputer05.pdf) provide the functionality of a computer
    - Instructure Set Architecture defines how CPU is controlled by the software. There are 3 main types:
        - register based
        - stack based (EVM)
        - accumulator based
    - a zkVM == combination of zk proof and a VM. It consists of two parts:
        - compiler that will compile high-level languages (C++/Rust) into intermediate (IR) expressions for the ZK system to perform
        - ISA (Instruction Architecture) => executes instructions about CPU operations & instructs the CPU to perform operations
- EVM architecture:
    - the opcodes need to interact with Stack, Memory and Storage
    - Stack => used for Stack access
    - Memory/Storage => accessed randomly
- zkEVM:
    - is a VM which recreates all existing EVM opcodes
    - acts as a state machine, proccesses state transitions from L2 txs
    - generates validity proofs that confirm the accuracy of the off-chain state change computations
    - architecture will have a sequencer and prover (or multiples of each), L1-L2 messaging capabilities and a contract on the L1
    - emulates all of EVM components and their interactions and create proofs that the interactions were correct
    - phases:
        - proof focused:
            - circuit creation
            - setup (proving/verification keys)
            - proof creation
            - proof aggregation
            - proof acceptance on L1 and verification
        - L2 aspects:
            - submit data to DA layer
            - allow L1-L2 messaging
            - provide escape hatch via forces txs
    - workflow:
        - receive tx
        - execute bytecode
        - make state changes and tx receipts
        - produce proof of correct execution (using zkEVM circuits w execution trace as input)
        - aggregate proofs for a bundle of txs and submit them to L1
        - submit data to DA layer
    - L1 relies on re-execution of SM. Bytecode is stored on storage. Txs are broadcasted accross as P2P network. For each tx, every node executed the bytecode (opcodes within it one by one) with tx as input. Each opcode has 3 sub-steps: read from stack/memory/storage, perform computations, write back the results to stack/memory/storage. 
    - L2 relies on validity proofs of zkEVM circuits. Bytecode also stored in storage. Txs are sent to a centralized zkEVM node. zkEVM generates succint proof demonstrating correct state transition. L1 contract verifies the proof and updates the state without re-executing the txs.
    - proofs must:
        - confirm correct bytecode is loaded
        - demonstrate opcodes are executed sequentially (without missing/skipping any of them)
        - verify correct execution of each opcode
    - taxonomy:
        - [intro](https://scroll.io/blog/zkEVM)
        - [overview](https://scroll.mirror.xyz/nDAbJbSIJdQIWqp9kn8J0MVS4s6pYBwHmK7keidQs-k)
        - [vitalik's view](https://vitalik.ca/general/2022/08/04/zkevm.html)
        - [circuits](https://github.com/privacy-scaling-explorations/zkevm-circuits)
        - [specs](https://github.com/privacy-scaling-explorations/zkevm-specs)
        - [explorations](https://github.com/privacy-scaling-explorations/)
        - [L2 tx life cycle](https://wiki.polygon.technology/docs/zkevm/protocol/l2-transaction-cycle-intro/)
    - proving system [overview](https://wiki.polygon.technology/docs/zkevm/zkProver/overview/)
        - the prover insures that all the rules for a tx to be valid are enforced, otherwise the proof would have no meaning
        - the circuit/proof is for any general contract
        - must check:
            - contract has been loaded correctly
            - tx signatures are correct
            - the state changes are correct
            - the execution proceeded correctly
        - proofs have 2 componets:
            - [state proof](https://github.com/privacy-scaling-explorations/zkevm-specs/blob/master/specs/state-proof.md) checks state/memory/stack ops have been performed correctly. Doesn't check for correct read/write location. Helps EVM proof to check all the random read-write access records are valid. Confirms the transition of state trie root is valid.
            - [EVM proof](https://github.com/privacy-scaling-explorations/zkevm-specs/blob/master/specs/evm-proof.md) checks the correct opcode is called at the correct time. Checks the validity of these opcodes. Confirms opcodes & state proof performed the correct operations
    - projects:
        - ranked on [L2Beat](https://l2beat.com/scaling/summary) / [Dune](https://dune.com/cryptokoryo/zk)
        - [Scroll](https://docs.scroll.io/en/technology/)
            - settlement layer: 
                - contains the bridge contract & rollup contract (rollup contract is also the verifier) (deployed on Ethereum)
                - provides data availability and ordering for the chanonical Scroll chain
                - verifies validity proofs
                - allows L1-L2 messaging
            - sequencing layer contains:
                - Execution Node:
                    - executes txs submitted to the Scroll sequencer / L1 bridge contract
                    - produces L2 blocks
                - Rollup Node:
                    - batches txs together
                    - posts txs data and block info for DA
                    - submits validity proofs to L1 for finality
            - proving layer consists of:
                - provers: generates the validity proofs what verify the correctness of L2 txs
                - coordinator: dispatches the proving tasks to provers and relays the proofs to Rollup Node
        - [Polygon](https://wiki.polygon.technology/docs/zkevm/) [zkEVM](https://mirror.xyz/msfew.eth/JJudP_Kf-IS6VhbF-qU0BUor1Ap6SFEb0TzYOHZ34Rc)
            - Sequencer reads txs from the pool
            - Sequencer orders and executes txs
            - executed txs are added to txs batch and Sequencer's local L2 State is updated
            - once a tx is added to the L2 State, it is broadcasted to all other nodes
            - fast finality can be achieved (potentially as soon as the sequencer has the tx)
            - L2 state is in a trusted state until the batch is committed to L1
        - [zkSync](https://era.zksync.io/docs/)
            - validator can't corrupt the state or steal funds (unlike sidechains)
            - users can force withdraw from rollup even if validator stops cooperating because data is available (unlike plasma)
            - not needed to monitor rollup blocks to prevent fraud (unlike payment channels or optimistic rollups)
        - [Starknet](https://docs.starknet.io):
            - [mindmap](https://github.com/0xAsten/Starknet-Tech-Stacks-Mindmap)
            - [architecture](https://david-barreto.com/starknets-architecture-review/)
            - [ecosystem](https://www.starknet-ecosystem.com/)
        - [Risc Zero](https://www.risczero.com/) - zkVM for general computation
        - [powder](https://docs.powdr.org/) - modular compiler to build zkVMs ([video](https://www.youtube.com/watch?v=P52kocriqFY))
        - [Polygon Miden](https://www.alchemy.com/overviews/polygon-zk-rollups) - 5000 tx/block => 1000 TPS at launch
    - proving systems [comparison](https://blog.celer.network/2023/08/04/the-pantheon-of-zero-knowledge-proof-development-frameworks/)
    - zkVM design [video](https://www.youtube.com/watch?v=tWJZX-WmbeY)
    - zk proofs [videos](https://www.youtube.com/playlist?list=PLS01nW3Rtgor_yJmQsGBZAg5XM4TSGpPs) / [course](https://zk-learning.org/)
    - arithmetic circuits:
        - represented as R1CS => can be transformed into polynomials => next stage in the creation of the proof
        - can be visualized as a number of addition and multiplication gates
        - not algorithms => more like hardware circuits
        - have constrains => satisfied if the correct input to the gates are supplied (i.e. the correct witness)
        - implemented using large tables => a gate is represented by a row in the table






## [3.4. What are ZK EVMs Part 2 (Universal Circuits) ](https://www.youtube.com/watch?v=d046QtF6fqM)

Q:
1. On L1 goes one proof which is aggregated from multiple proofs. Where do these proofs come from? How are they split. What do they represent individually?

1.
