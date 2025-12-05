# Rollupstr

Rollupstr is an isolation-first layer built around scripts and their local state, anchored to Bitcoin with a single 32-byte hash per batch, and using open networks like Nostr alongside external payment systems such as Lightning, Ark v2, Fedimint, or on-chain Bitcoin for value movement.

It is not a global virtual machine and not a shared account ledger.

Each user only downloads and replays the script states they actually care about — their own scripts and any explicit dependencies — and proves those states back to Bitcoin through compact Merkle proofs.


---

Core idea

Rollupstr is built on the idea of isolated script states.

A script is a small, self-contained state machine.
It defines its own ID, its own state or state root, a sequence of state transitions, optional dependencies on other scripts, and optional references to external payment proofs.

There is no global world state to sync.
Your client only interacts with the minimal dependency graph required for your activity.


---

Data flow

Rollupstr separates responsibilities across three layers:

1. State and logic — stored in scripts and their transitions.


2. Transport and availability — Nostr or any broadcast layer for discovering transitions.


3. Settlement and finality — Bitcoin anchoring each batch with a 32-byte commitment.



Payments happen externally and are not tracked inside Rollupstr.


---

Scripts and transitions

A script transition typically includes the script ID, the previous state hash or version, the new state hash or diff, references to dependent script states, and optional external payment proofs.

Aggregators collect transitions from Nostr, order them, commit them into a Merkle structure, and publish the Merkle root to Bitcoin.
The transitions stay off-chain; Bitcoin only commits the hash.


---

What a client actually does

A Rollupstr client does not download or replay a global rollup.

It only:

1. Selects a script to interact with.


2. Fetches the script’s transitions, dependency transitions, Merkle proofs, and the Bitcoin anchors containing the batch roots.


3. Replays only that small local subgraph.


4. Verifies script logic, dependency state, Merkle inclusion, Bitcoin anchoring, and external payment proofs.



Your entire node consists of a few scripts, their transitions, their proofs, and the Bitcoin anchors. Nothing global.


---

External payments

Rollupstr does not become a balance ledger.
Value moves in systems that supply client-verifiable receipts, such as Lightning, Ark v2, Fedimint, or on-chain Bitcoin.

Scripts may require those proofs:

“This update is valid only if a Lightning invoice preimage is provided.”
“This action requires a valid Ark spend proof.”
“This transition depends on an on-chain payment receipt.”

Rollupstr never tracks full payment histories.
It only validates the provided proofs according to the script rules.


---

Isolation and dependencies

Each script is an isolated island of state.

It can operate independently or declare explicit dependencies on other scripts.
When a script depends on another, the client fetches and verifies only those transitions, forming a small verifiable state graph.

Users never download unrelated scripts or unrelated activity.


---

Anchoring and proofs

Aggregators batch script transitions into a Merkle tree and publish the root to Bitcoin.

To verify a script state, the client only needs the transitions for that script, transitions for its dependencies, the Merkle proofs, and the Bitcoin transaction containing the root.
From these, the client reconstructs the script state and its anchoring to Bitcoin.


---

Mental model

Rollupstr is not a blockchain.

It is a structure for verifiable, local script states that anchor collectively to Bitcoin without global replication.

Many scripts, each with its own state machine.
Optional dependencies forming small graphs.
External systems handle value.
Bitcoin provides the anchor.

Users verify only the part of the world they touch — nothing more.
