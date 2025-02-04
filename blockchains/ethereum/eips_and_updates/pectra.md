## [DRAFT] security considerations for **[pectra](https://eips.ethereum.org/EIPS/eip-7600)** (early 2025)

<br>

### ✅🔐🏋🏻‍♀️ **[eip-2537: precompile for bls12-381 curve operations, by a. vlasov et al.](https://eips.ethereum.org/EIPS/eip-2537)**

<br>

  - adds bls signature verification and operations over `bls12-381`, a cryptographic primitive allowing to get `120+` bits of security for operations over pairing friendly curve (compared to the existing `bn254` precompile that only provides `80` bits of security)
    
<p align="center">
<img src="https://github.com/user-attachments/assets/8107c4b3-5bd0-4a79-b1eb-4eb6c771c022" width="80%"/>
</p>

  - ddos protection:
    - *a sane implementation of this eip should not contain potential infinite loops (it is possible and not even hard to implement all the functionality without while loops) and the gas schedule accurately reflects the time spent on computations of the corresponding function (precompiles pricing reflects an amount of gas consumed in the worst case where such a case exists)*

<br>

---

### ✅🔐🤝🏋🏻‍♀️ **[eip-6110: supply validator deposits on chain, by m. kalinin et al.](https://eips.ethereum.org/EIPS/eip-6110)**

<br>

  - appends validator deposits to the el block structure, shifting the responsibility of deposit inclusion and validation to the el and removes the need for deposit
  - faster for validators to deposit their eth (12 hours -> 30 min)
  - removes the need for deposit voting from the cl, reducing the complexity of client software design, and contributing to the security of the deposits flow

<p align="center">
<img src="https://github.com/user-attachments/assets/d7dfb8cd-f496-4087-ac16-72384a2a0204" width="80%"/>
</p>

<br>

---

### ✅🔐🤝🏋🏻‍♀️ **[eip-7002: execution layer triggerable exits, by djrtwo et al.](https://eips.ethereum.org/EIPS/eip-7002)**

<br>

  - improves ux for validators, giving them more flexibility
  - allows validators to trigger exits and partial withdrawals via their el (`0x01`) withdrawal credentials (e.g., enabling more trustless staking pool designs)
  - these new execution layer exit messages are appended to the el block and then processed by the cl

<p align="center">
<img src="https://github.com/user-attachments/assets/e0b79fb2-4665-4ac2-8adf-bf455773df16" width="80%"/>
</p>

  - security consideration on fee overpayment:
    - *calls to the system contract require a fee payment defined by the current contract state. overpaid fees are not returned to the caller*
    - *it is not generally possible to compute the exact required fee amount ahead of time*
    - *using an eoa to request withdrawals will always result in overpayment of fees*
    - *there is no way for an eoa to use a wrapper contract to request a withdrawal*
    - *and even if a way existed, the gas cost of returning the overage would likely be higher than the overage itself*
    - *if requesting withdrawals to an eoa through the system contract is desired, we recommend that users perform transaction simulations to estimate a reasonable fee amount to send*

<br>

---

### ✅🔐🤝 **[eip-7251: increase the `MAX_EFFECTIVE_BALANCE`, by m. neuder et al.](https://eips.ethereum.org/EIPS/eip-7251)**

<br>

  - biggest ux improvement for validators
  - raises the validator stake limit (maximum effective balance from 32 -> 2048 eth, with reward compounding)
  - potentially can reduce the number of inactive nodes and possibly improving the network efficiency
  - security:
    - *proposer selection is already weighted by the ratio of their effective balance to `MAX_EFFECTIVE_BALANCE`*
    - *due to the lower probabilities, this change will slightly increase the time it takes to calculate the next proposer index*

<p align="center">
<img src="https://github.com/user-attachments/assets/ae1c8286-ef18-457c-84ba-0df620acaf9e" width="80%"/>
</p>

<br>

---

### ✅🔐🤝 **[eip-7549: move committee index outside attestation, by dapplion](https://eips.ethereum.org/EIPS/eip-7549)**

<br>

  - makes the aggregation of validator votes (attestation) in blocks more efficient, reducing networking load and saving node bandwidth
  - this eip introduces backward incompatible changes to the block validation rule set on the consensus layer and must be accompanied by a hard fork
  - because the on chain `Attestation` container changes, attestations from the prior fork can't be included into post-electra blocks, therefore the first block after the fork may have zero attestations

<p align="center">
<img src="https://github.com/user-attachments/assets/837baf42-99d4-47f0-80b9-1bd19986a5a6" width="80%"/>
</p>

<br>

---

### ✅🔐🤝🏋🏻‍♀️ **[eip-7702: set eoa account code, by vub et al.](https://eips.ethereum.org/EIPS/eip-7702)**

<br>

  - add a new transaction type that adds a list of `[chain_id, address, nonce, y_parity, r, s]` authorization tuples, improving the functionality of crypto wallets by giving them smart contract properties (the so called "account abstraction")

<p align="center">
<img src="https://github.com/user-attachments/assets/8dd4c850-0b5e-4ac4-b44d-2c59e2bc8968" width="80%"/>
</p>

  - more usability in crypto, enhanced security features:
    - batching multiple operations from the same user into one atomic transaction (e.g., erc-20's approval + spending)
    - sponsorship (an account can pay for a transaction on behalf of another account)
    - privilege de-escalation where users can sign sub-keys, giving them specific permissions that are much weaker than global access to the account (e.g., a permission to spend erc-20 but not eth, or spend a percentage of balance per day, or interact only with a specific application)
   
<p align="center">
<img src="https://github.com/user-attachments/assets/3cb0f48b-88aa-4844-9a46-ede10e09837e" width="80%"/>
</p>

  - introduces a new transaction, the setcode tx, very similar to eip-1559 txs, with an additional authorization list elements ("authorizing some code to live into your account" through creating by template)

<p align="center">
<img src="https://github.com/user-attachments/assets/76a0d234-61d9-4617-bc3e-e3fe8903bb40" width="80%"/>
</p>

  - e.g., gas fees could be outsourced to services to pay on another erc-20 token

<p align="center">
<img src="https://github.com/user-attachments/assets/6d712fe2-309f-4ce8-bd18-b4cf253fdbac" width="80%"/>
</p>

<br>

#### main security challenges: full account abstraction introduces ddos vectors

- the main challenges are efficient block building and DoS-resilient p2p mempool (unused gas as a vector can become a side channel attack)
- example:
    1. suppose an attack implement an attacker and this account depends on a flag stored in a singleton smart contract
    2. every account using this implementation looks at the same flag to determine the transaction validity (and also flips the flag)
    3. the attacker then sends thousands of such transactions, and every time such transaction gets included, it immediately invalidates all the other ones, so they can't be included in the chain and need to be dropped without paying gas (since they are not valid)
    4. this can escalate to a point where the nodes cannot do any useful work
- the mitigation can done by separating validation from execution so that the mempool protocol only propagates compliant transactions
- block builders must be able to validate each transaction independently (parallelizing their validation)
  - if validation accessible data overlaps, transactions can invalidate each other causing another DoS attack vector is the mempool is filled with mutually exclusive txs
 
<p align="center">
<img src="https://github.com/user-attachments/assets/f273e78f-1f51-41c4-adbb-3e2ec48ce630" width="80%"/>
</p>

<br>

---

### ✅🏋🏻‍♀️ **[eip-7685: general purpose execution layer requests, by lightclient](https://eips.ethereum.org/EIPS/eip-7685)**

<br>

  - boosts the interoperability between the execution and the consensus layer (helping with surge demand on the execution layer) by defining a general purpose framework for storing contract-triggered requests:
    - extends the execution header with a single field to store the request information
    - requests are later on exposed to the consensus layer, which then processes each one
  - more efficient way to code, test, and implement execution triggered requests such as eip-6110 and eip-7002
  - note on "exec" concern:
    - *a request’s validity can often not be fully verified at the execution layer*
    - *this is why they are referred to merely as "requests": they do not carry the authority on their own to unilaterally catalyze an action*
    - *we expect the system contracts to perform whatever validation is possible by the el and then pass it on to the cl for further validation*

<br>

---

### ✅🏋🏻‍♀️ **[eip-2935: save historical block hashes from state, by vub et al.](https://eips.ethereum.org/EIPS/eip-2935)**

<br>

  - store last `HISTORY_SERVE_WINDOW` historical block hashes in the storage of a system contract as part of the block processing logic
  - increases amount of data from past blocks that can be stored on new blocks
  - sets the stage for verkle tree
  - improves solo staking ux: enabling stateless validator clients, allowing staking nodes to run with very little hard disk space and quick sync
  - security:
    - *having contracts (system or otherwise) with hot update paths (branches) poses a risk of "branch" poisioning attacks where attacker could sprinkle trivial amounts of eth around these hot paths (branches)*
    - *but it has been deemed that cost of attack would escalate significantly to cause any meaningful slow down of state root updates*

<br>

---

### ✅ **[eip-7692: evm object format meta, by a. beregszaszi et al.](https://eips.ethereum.org/EIPS/eip-7692)**

<br>

  - more zk friendly!
  - adds a bunch of evm object format for smart contract deployment and execution efficiency, including optimized code validation, better function handling, more efficient data access instructions:
    - **[eip-3540](https://eips.ethereum.org/EIPS/eip-3540)**:
      - introduces an extensive container format for the evm with a once-off validation at deploy time
      - `magic, version, (section_kind, section_size_or_sizes)+, 0, <section contents>`
    - **[eip-3670](https://eips.ethereum.org/EIPS/eip-3670)**:
      - validates eof bytecode for correctness at the time of deployment
    - **[eip-4200](https://eips.ethereum.org/EIPS/eip-4200)**:
      - `RJUMP`, `RJUMPI`, and `RJUMPV` instructions with a signed immediate encoding the jump destination
    - **[eip-4750](https://eips.ethereum.org/EIPS/eip-4750)**:
      - individual sections for functions with `CALLF` and `RETF` instructions
    - 🔐 **[eip-5450](https://eips.ethereum.org/EIPS/eip-5450)**:
      - deploy-time validation of stack usage for eof functions
      - security: introduces extended validation of eof code sections to guarantee that neither stack underflow nor overflow can happen during execution of validated contracts)
    - **[eip-6206](https://eips.ethereum.org/EIPS/eip-6206)**:
      - introduces instruction for chaining function calls
    - **[eip-7480](https://eips.ethereum.org/EIPS/eip-7480)**:
      - instructions to read data section of eof container (clear separation between code and data from eof1)
      - *accessing immutable values and contract payloads efficiently*
    - **[eip-663](https://eips.ethereum.org/EIPS/eip-663)**:
      - introduces additional instructions for manipulating the stack which allows accessing the stack at higher depths
      - the evm stack is fixed at 1024 items and most implementations keep that in memory at all times - this change increases the number of stack items accessible via single instruction
      - *helping compilers avoid "stack too deep" errors*
    - 🔐 **[eip-7069](https://eips.ethereum.org/EIPS/eip-7069)**:
      - revamped `CALL` intrusctions by introducing `EXTCALL`, `EXTDELEGATECALL`, `EXTSTATICALL`, with simplified semantics
      - *addressing gas introspection and preserving space for address space expansion*
      - security: when implemented in eof - where the `GAS` opcode and the original `CALL` operations are removed - existing out of gas attacks will be slightly more difficult, but not entirely prevented as transactions can still pass in arbitrary gas values
    - **[eip-7620](https://eips.ethereum.org/EIPS/eip-7620)**:
      - introduces `EOFCREATE` and `RETURNCONTRACT` instructions
      - *most code introspection issues are contract creation issues*
    - 🔐 **[eip-7620](https://eips.ethereum.org/EIPS/eip-7698)**:
      - deploys eof contracts using creation transactions
      - *how eof contracts are introduced into the blockchain*
      - security: external unverified code (e.g., check for correct price, dos)


<p align="center">
<img src="https://github.com/user-attachments/assets/14816acb-4679-4327-90a7-8b3a7d42c54c" width="80%"/>
</p>

<br>

---

### ✅🤝🏋🏻‍♀️ **[eip-7742: uncouple blob count between cl and el, by a. stokes](https://eips.ethereum.org/EIPS/eip-7742)**

<br>

  - extends functionalities from blobs
  - execution layer no longer verifies data blobs's maximum value and instead gets this value dynamically from the consensus layer

<br>

---

### ✅🤝 **[eip-7594: peerdas - peer data availability sampling, by djrtwo et al.](https://eips.ethereum.org/EIPS/eip-7594)**

<br>

  - allows beacon nodes to perform data availability sampling, improving how da is handled across the network
  - crucial feature for layer 2s (making them more efficient and cost-effective)
  - compare to celestia (tba)
 
<br>

---

### 🟡 **[eip-7623: increase calldata cost, by t. wahrstätter et al.](https://eips.ethereum.org/EIPS/eip-7623)**

<br>

  - increases the calldata cost for transactions (increase the cost of calldata to 10/40 gas for transactions that do not exceed a certain threshold of gas spent on evm operations)
  - highligting data availability

<br>

---

### 🟡 **[eip-7762: increase `MIN_BASE_FEE_PER_BLOB_GAS`, by m. resnick](https://eips.ethereum.org/EIPS/eip-7762)**

<br>

  -  speeds up discovery on blob space
  -  security: "rollups that use blobs as da will need to update their posting strategies"

<br>

----

### cool resources

<br>

* **[pectra retrospective at eth magickians, by t. beiko et al. (2025)](https://ethereum-magicians.org/t/pectra-retrospective/22637)**
* **[what's going into the pectra upgrade?, by c. kim (2024)](https://www.youtube.com/watch?v=ufIDBCgdGwY)**
* **[eip-7702: a technical deep dive, by lightclient (2024)](https://www.youtube.com/watch?v=_k5fKlKBWV4)**
* **[exploring eip-7702, by y. weiss (2024)](https://www.youtube.com/watch?v=63Wd5mPla-M)**
* **[native aa in pectra - combining eof, eip-7702, by a. forshtat (2024)](https://www.youtube.com/watch?v=FYanFF-yU6w)**
* **[evm object format (eof) - history and motivation, by d. ferrin (2024)](https://www.youtube.com/watch?v=X2mlptWzphc)**
* **[evm object format (eof) - managing the bytecode chaos, by a. murashkin (2024)](https://www.youtube.com/watch?v=WKVgCoNp39g)**
* **[unpacking eof: applications, by n. urisevic et al. (2024)](https://www.youtube.com/watch?v=OsKyVPdpJgI)**
* **[eip-7251, maximum effective balance overview, by p. harris (2024)](https://www.youtube.com/watch?v=EwW6dNi9VCY)**


