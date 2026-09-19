# TRU Smart Contracts

> **A simple guide to what TRU contracts are, what the working contract types do, and how to use them.**
>
> This README documents **working/current TRU contract capabilities only**. It intentionally leaves out retired/legacy bridge surfaces and roadmap-only contract ideas that are not ready for users.

---

# 🧠 TRU CONTRACTS, EXPLAINED SIMPLY

## What is a smart contract?

A TRU smart contract is a transaction with rules attached to it.

Think of normal TRU like money in a wallet:

```text
YOU
 │
 └──► send TRU to another address
```

A contract adds a rule in the middle:

```text
YOU
 │
 └──► CONTRACT
          │
          ├── wait until a certain time
          ├── require a secret
          ├── require 2 of 3 signatures
          ├── require a vote/state update
          └── require some other valid condition
                    │
                    ▼
                 TRU moves
```

The important part is that **the blockchain enforces the rule**.

The website, wallet, CLI, or explorer can help you build and inspect the contract, but they do not get to ignore the chain rules.

---

# ✅ WHAT WORKS TODAY

TRU currently has these working contract/script capabilities:

| Contract | Simple meaning | Good for |
|---|---|---|
| **Time Lock** | TRU cannot move until the lock matures | Delayed payments, scheduled release |
| **Hash Lock** | TRU moves only when the correct secret is revealed | Secret-based claims, swap building block |
| **Oracle Lock** | A deterministic oracle/feed condition controls execution | Data-triggered contracts |
| **Stateful Key / Value** | Stores state that can be updated over time | On-chain app state |
| **Voting** | Stores a poll and updates vote state | On-chain voting |
| **Token Issuer** | A contract that issues token units under fixed rules | Controlled token issuance |
| **Multisig / Escrow** | Any 2 of 3 approved keys can release the TRU | Escrow, shared control |
| **HTLC / Atomic Swap** | Secret claim path + timeout refund path | Noncustodial atomic swaps |
| **OP_RETURN** | Permanently publishes data with no spendable output value | Proofs, references, metadata |
| **Custom Script** | Advanced approved script path | Developers / specialized logic |
| **MagicLock** | Specialized target/secret locking facility | Advanced locking experiments |

> **Not included here:** legacy/retired bridge surfaces and future roadmap contract ideas.  
> If a feature is not ready for normal use, it does not belong in this guide.

---

# 🚦 THE ONE CONTRACT RULE TO REMEMBER

Creating a transaction is not the same thing as confirming it.

The normal lifecycle is:

```text
CREATE
  ↓
MEMPOOL ACCEPTED
  ↓
WAIT FOR A BLOCK
  ↓
CONFIRMED
  ↓
USE / CALL / REDEEM / UPDATE
  ↓
MEMPOOL ACCEPTED
  ↓
WAIT FOR A BLOCK
  ↓
CONFIRMED RESULT
```

If TRU Core says:

```text
MEMPOOL ACCEPTED
```

that means:

> “The node accepted the transaction and is waiting for it to be mined.”

It does **not** mean:

> “This is already permanent confirmed chain state.”

For contract work, get used to this rhythm:

**Create → Confirm → Act → Confirm → Verify**

---

# 🪙 TRU AND TRU ATOMS

Human-facing screens normally show TRU:

```text
1 TRU
4 TRU
12.50000000 TRU
```

Internally the chain uses exact integer atomic units:

```text
1 TRU = 100,000,000 TRU atoms
1 TRU atom = 0.00000001 TRU
```

You usually do not need to think about atoms.

The main exception is **Token Issuer**, because its exchange-rate rule is defined in token units per TRU atom. That is explained in its section below.

---

# 🧭 WHERE TO FIND CONTRACTS

In TRU Core, the normal contract entry point is:

```text
Main Menu
  → Create Contract
```

Current builds expose the contract choices by name.

**Use the contract name, not just a menu number.** Menu numbers are a user-interface detail and can move as TRU Core evolves.

You can also use the Contract Vault / contract listing surfaces to inspect confirmed contracts.

On the TRU Block Explorer, open:

```text
Contracts
```

The Contract Registry lets you:

- search by contract name;
- search by creator/address;
- search by transaction ID;
- search by block;
- filter by protocol;
- filter active/released contracts;
- sort contracts;
- open the full contract inspector.

---

# 🔐 1. TIME LOCK


A Time Lock is a safe with a clock on it.

You can put TRU inside now, but it cannot be redeemed until the chain says the lock has matured.

```text
TRU
 │
 ▼
┌───────────────┐
│   TIME LOCK   │
│               │
│  too early?   │──► NO
│               │
│  mature?      │──► REDEEM
└───────────────┘
```

## What is it useful for?

Examples:

- delayed payments;
- scheduled releases;
- escrow-like waiting periods;
- “do not spend before this time” rules;
- a building block for HTLCs.

## Create one

In TRU Core:

```text
Create Contract
  → Time Lock
```

Then:

1. Enter the contract name/label.
2. Enter how much TRU you want to lock.
3. Enter the unlock/maturity information requested by Core.
4. Review the transaction.
5. Broadcast it.
6. Save the TXID/outpoint shown in the receipt.
7. Wait for the creation transaction to be mined.
8. Check the Contract Vault or explorer.

You should now see the TRU as locked.

## Redeem it

Before maturity:

```text
REDEEM
  ↓
REJECTED
```

After maturity:

1. Open the confirmed Time Lock.
2. Choose the Time Lock redemption action.
3. Enter the requested destination/authorization information.
4. Broadcast the redemption transaction.
5. Wait for it to be mined.
6. Verify the original locked output is now spent.
7. Verify the recipient output is confirmed.

## Important

Do not test a Time Lock redemption before the **creation transaction itself is confirmed** and assume any error means the time rule worked.

First prove the locked output exists on-chain. Then test maturity.

---

# 🔑 2. HASH LOCK

## ELI5

A Hash Lock is a safe that opens with a secret.

You do **not** put the secret itself in the lock.

Instead, the contract stores a cryptographic fingerprint of the secret.

```text
SECRET
  │
  ▼
HASH
  │
  ▼
stored in contract
```

Later:

```text
candidate secret
      │
      ▼
    hash it
      │
      ├── matches → unlock
      └── wrong   → reject
```

The secret is commonly called a **preimage**.

## What is it useful for?

- secret-based redemption;
- proofs that someone knows a value;
- atomic-swap construction;
- conditional payments.

## Create one

```text
Create Contract
  → Hash Lock
```

Then:

1. Choose and safely record your secret/preimage.
2. Enter the hash/preimage information requested by Core.
3. Enter the amount of TRU to lock.
4. Create and broadcast the transaction.
5. Save the contract TXID/outpoint.
6. Wait for confirmation.
7. Verify it appears in the Contract Vault/explorer.

## Redeem it

1. Open the confirmed Hash Lock.
2. Choose the Hash Lock redemption action.
3. Provide the correct preimage.
4. Enter the requested recipient/authorization information.
5. Broadcast.
6. Wait for confirmation.
7. Verify the locked output is spent.

A wrong preimage should be rejected.

## Safety tip

Treat the preimage like a password until you intend to reveal it.

Once a preimage is published in a successful claim transaction, assume it is public forever.

---

# 🌐 3. ORACLE LOCK

## ELI5

An Oracle Lock is a contract that asks:

> “Does the trusted data source say the required thing happened?”

Example idea:

```text
Oracle key: weather.springfield
Expected:   RAIN

actual authoritative value = RAIN
              │
              ▼
         condition passes
```

The chain does not magically know weather, sports, prices, or sensor readings by itself.

An oracle/feed path supplies deterministic data that the contract can use.

## What is it useful for?

- external-data conditions;
- sensor-driven applications;
- event outcomes;
- deterministic feed-based release logic.

## Create one

```text
Create Contract
  → Oracle-Locked
```

Then:

1. Enter the oracle/feed key.
2. Enter the expected value/condition requested by Core.
3. Enter the amount of TRU.
4. Create and broadcast.
5. Wait for confirmation.
6. Verify the contract is visible.

## Resolve/use it

1. Make sure the **authoritative oracle/feed path** contains the required value.
2. Invoke the Oracle-Locked action.
3. Broadcast the resulting transaction.
4. Wait for confirmation.
5. Verify the resulting spend/state.

## Important

A value typed into a browser or shown in a local UI is not automatically authoritative oracle state.

The contract must use the data path recognized by the protocol.

---

# 🗂 4. STATEFUL KEY / VALUE

## ELI5

Stateful Key/Value is like a tiny on-chain notebook.

Example:

```text
status = "NEW"
```

Later it can become:

```text
status = "SHIPPED"
```

and later:

```text
status = "DELIVERED"
```

The contract is still the **same logical contract** even though its current on-chain state moves forward.

## The two IDs to understand

TRU uses two important ideas:

### Stable Root

Think of this as the contract's birth certificate.

```text
Stable Root = permanent identity
```

It does not change just because the state changes.

### Live Anchor

Think of this as the latest page in the notebook.

```text
Live Anchor = current confirmed state
```

When you successfully update the contract:

```text
Stable Root
   stays the same

Live Anchor
   moves forward
```

## Create one

```text
Create Contract
  → Stateful Key/Value
```

Then:

1. Enter the contract name.
2. Enter the initial key/value state.
3. Enter any requested TRU contract value.
4. Create and broadcast.
5. Wait for confirmation.
6. Record the Stable Root.

## Read the state

Use the current:

```text
Query State
```

surface in TRU Core.

## Update the state

Use:

```text
Update K/V
```

Then:

1. Select/reference the contract requested by Core.
2. Enter the replacement key/value.
3. Broadcast.
4. Wait for confirmation.
5. Query it again.

Expected result:

```text
Stable Root = SAME
Live Anchor = NEW
State       = NEW VALUE
```

A stale old state should not become a second valid current state.

---

# 🗳 5. VOTING

## ELI5

Voting is a stateful contract that keeps a poll on-chain.

```text
QUESTION
   │
   ├── Option A
   ├── Option B
   └── Option C
```

Votes update the contract state under deterministic rules.

## What is it useful for?

- community polls;
- project decisions;
- game/community voting;
- auditable tally experiments.

## Create a vote

```text
Create Contract
  → Voting Contract
```

Then:

1. Enter the poll/contract name.
2. Enter the allowed choices/options.
3. Enter the other voting parameters requested by Core.
4. Broadcast.
5. Wait for confirmation.

## Cast a vote

Use the current:

```text
Vote
```

surface.

1. Select the confirmed voting contract.
2. Select a valid option.
3. Broadcast the vote.
4. Wait for confirmation.

## Read results

Use:

```text
Voting Results
```

The chain should reject invalid options and actions that violate the contract's voting rules.

Because Voting is stateful, the Stable Root stays the same while the current live state advances.

---

# 🏭 6. TOKEN ISSUER

## ELI5

A Token Issuer is a vending machine on-chain.

You define rules such as:

```text
Token Name
Exchange Rate
Maximum Supply
```

and the contract tracks how many units have been issued.

```text
TRU/value enters
      │
      ▼
TOKEN ISSUER
      │
      ├── checks rate
      ├── checks max supply
      └── issues allowed token units
```

## Important: the issuer is not the token

These are different things:

```text
TOKEN
  → Token Registry / Token Vault

TOKEN ISSUER CONTRACT
  → Contract Registry
```

A Token Issuer can later create/issue token units, but the issuer contract itself is not an NFT, SFT, FT, or NCFT.

## Create one

```text
Create Contract
  → Token Issuer
```

Core asks for information such as:

```text
contract/token name
exchange rate
maximum supply
TRU contract value
```

Then:

1. Review the values carefully.
2. Broadcast.
3. Wait for confirmation.
4. Open the confirmed issuer in the Contract Vault.
5. Verify supply/rate information.

## Mint/purchase from it

Use the current Token Issuer mint/purchase surface.

For each issuance:

1. Use the confirmed current issuer state.
2. Enter the requested amount/value.
3. Broadcast.
4. Wait for confirmation.
5. Verify:
   - total issued;
   - remaining supply;
   - resulting token balance.

The contract must not issue past its configured maximum supply.

## ⚠️ Exchange-rate detail

The V1 issuer rate is defined as:

```text
token units per TRU atom
```

and:

```text
1 TRU = 100,000,000 TRU atoms
```

So always read the confirmation screen carefully before creating an issuer.

---

# 👥 7. MULTISIG / ESCROW — 2 OF 3

## ELI5

Multisig is a safe with three keys where **any two keys are required**.

```text
KEY A ─┐
KEY B ─┼──► need ANY 2 ──► RELEASE
KEY C ─┘
```

One person alone cannot release the funds.

## What is it useful for?

- buyer/seller/mediator escrow;
- business treasury control;
- family/shared funds;
- backup-key arrangements;
- organizations where no single person should control the money.

## Create an escrow

```text
Create Contract
  → Multisig / Escrow (2-of-3)
  → Create
```

You need three compressed secp256k1 public keys.

At a participant prompt, current Core can use:

```text
me
```

for the current wallet's compressed public key when appropriate.

Then:

1. Enter the three participant public keys.
2. Enter the TRU amount.
3. Review and confirm.
4. Save the funding TXID.
5. Save the contract outpoint.
6. **Wait for the funding transaction to be mined before signing.**
7. Verify the Contract Vault shows:
   - `MULTISIG / ESCROW V1`
   - `UNSPENT`

## Create signatures

Each signer independently chooses:

```text
Multisig / Escrow
  → Sign
```

Both signers must agree on the exact same:

```text
funding transaction
release recipient
```

The signing step creates a local signature package.

Conceptually:

```text
Signer A ──► signature A
Signer B ──► signature B

signature A + signature B
          │
          ▼
       FINALIZER
          │
          ▼
     redemption TX
```

A signing package is **not itself a broadcast transaction**.

## Redeem

Choose:

```text
Multisig / Escrow
  → Redeem
```

Provide the required funding information, recipient, two participant public keys, and their corresponding signatures.

The finalizer reconstructs and verifies the signatures before building the spend.

After broadcast:

1. Wait for confirmation.
2. Verify the escrow is now `SPENT`.
3. Verify the recipient received the released amount minus the normal transaction fee.

## Important

A public key is not enough to sign.

The signing wallet must actually contain the corresponding private key.

Never share the private key itself.

---

# ⚛️ 8. HTLC / ATOMIC SWAP

## ELI5

HTLC stands for:

**Hashed Timelock Contract**

It combines two ideas you already saw:

```text
HASH LOCK
+
TIME LOCK
=
HTLC
```

An HTLC has two possible doors.

### Door 1 — Claim

```text
correct secret/preimage
        +
authorized claimant
        │
        ▼
      CLAIM
```

### Door 2 — Refund

If nobody successfully claims before the timeout:

```text
timeout matures
      +
refund authorization
      │
      ▼
    REFUND
```

That makes HTLCs useful for **noncustodial atomic swaps**.

## What does “atomic swap” mean?

Imagine Alice has TRU and Bob has another supported asset.

They want to trade without giving custody of both assets to a middleman.

Both sides create matching conditional contracts using the same secret hash.

Simplified:

```text
ALICE'S SIDE                    BOB'S SIDE

TRU locked                      other asset locked
with HASH X                     with HASH X
     │                               │
     └──────── same secret ──────────┘
```

When the secret is used to claim one side, it becomes available for the matching claim on the other side.

If the swap does not finish correctly, timeout/refund logic provides an exit path.

## Basic TRU-side flow

Use:

```text
Create Contract
  → HTLC / Atomic Swap
```

The current HTLC flow uses the concepts:

```text
amount
hash commitment
claim/recipient identity
refund identity
timeout
```

Then:

1. Create the HTLC.
2. Broadcast it.
3. Wait for the funding transaction to confirm.
4. Verify the HTLC in the contract/explorer surfaces.

### Claim path

The claimant provides the required:

```text
funding contract/outpoint
recipient information
correct preimage
required authorization
```

The claim is broadcast and confirmed.

### Refund path

If the claim did not happen and the timeout has matured, the refund party uses the refund action and required authorization.

The refund is then broadcast and confirmed.

## Why this matters

An HTLC is **not a bridge** and it does not require creating a wrapped TRU token.

It is a conditional exchange primitive.

The broader TRU swap system can use HTLCs to coordinate two-chain swaps while the parties retain control of their respective keys.

## Safety tips

- Test with tiny amounts first.
- Back up the secret/preimage safely.
- Never reuse a secret carelessly across unrelated swaps.
- Confirm funding before attempting claim/refund.
- Understand the timeout before funding.
- Do not assume a browser message equals chain confirmation.

---

# 📝 9. OP_RETURN

## ELI5

OP_RETURN is a permanent note attached to the blockchain.

```text
"Hello TRU"
     │
     ▼
OP_RETURN
     │
     ▼
on-chain data
```

It is intentionally **not spendable TRU**.

## What is it useful for?

- document hashes;
- proof references;
- application metadata;
- IDs;
- short permanent messages;
- anchoring external records.

## Create one

```text
Create Contract
  → OP_RETURN
```

Then:

1. Enter the data/payload.
2. Review it carefully.
3. Create and broadcast.
4. Wait for confirmation.
5. Inspect the transaction on the explorer.

The OP_RETURN output should carry **zero spendable TRU value**.

## Important

On-chain data is public and permanent.

Do not put:

- passwords;
- private keys;
- seed phrases;
- confidential personal information

into OP_RETURN.

---

# 🧩 10. CUSTOM SCRIPT

## ELI5

Custom Script is the advanced lane.

Instead of choosing a named contract like Time Lock or Voting, a developer can use a script that matches TRU's approved structural policy.

This is **not**:

> “Run anything I want.”

It is:

> “Use a script that the current TRU policy recognizes as valid and relayable.”

## Who should use it?

Developers who understand:

- TRU transaction scripts;
- opcodes;
- spend conditions;
- transaction construction;
- mempool policy;
- block validation.

## Basic flow

```text
Create Contract
  → Custom Script
```

Then:

1. Build an approved/canonical script.
2. Enter any permitted TRU value.
3. Review the exact script.
4. Broadcast.
5. Wait for confirmation.
6. Verify classification and spend behavior.

## Important

Do not use Custom Script to try to bypass the rules of a named contract family.

TRU's relay policy uses structural recognition rather than simply trusting a label.

---

# ✨ 11. MAGICLOCK

## ELI5

MagicLock is a specialized TRU script facility with its own lock/unlock path.

It is not treated as one of the normal stateful Stable-Root/Live-Anchor families.

Think of it as an advanced puzzle lock.

## Create

Use the dedicated MagicLock/script facility in the wallet.

Typical flow:

1. Choose the target-prefix condition.
2. Enter the TRU amount.
3. Preserve any required optional secret material.
4. Broadcast.
5. Wait for confirmation.

## Unlock

1. Open the matching MagicLock unlock action.
2. Provide the required target/secret authorization.
3. Broadcast.
4. Wait for confirmation.
5. Verify the locked UTXO is spent.

---

# 🧬 STATEFUL CONTRACTS: WHY “STABLE ROOT” MATTERS

You will see these terms in TRU:

```text
Stable Root
Live Anchor
```

Here is the easy version.

Imagine a car.

Its VIN stays the same for the life of the car:

```text
Stable Root = VIN
```

But its current odometer reading changes:

```text
Live Anchor = current odometer reading
```

For a TRU stateful contract:

```text
CREATE
  │
  ▼
Stable Root = ABC
Live Anchor = ABC
State = 1

UPDATE
  │
  ▼
Stable Root = ABC
Live Anchor = DEF
State = 2

UPDATE
  │
  ▼
Stable Root = ABC
Live Anchor = XYZ
State = 3
```

The contract did not become three separate contracts.

It is one contract with three historical states.

That distinction is important for:

- Stateful Key/Value;
- Voting;
- Token Issuer;
- other stateful contract families.

---

# 🔎 HOW TO VERIFY A CONTRACT

Do not rely on only one screen.

A good verification routine is:

```text
1. CREATE
2. SAVE TXID / OUTPOINT
3. WAIT FOR CONFIRMATION
4. OPEN CONTRACT VAULT
5. OPEN BLOCK EXPLORER
6. VERIFY STATUS / VALUE / CREATOR
7. PERFORM ACTION
8. WAIT FOR CONFIRMATION
9. VERIFY FINAL STATE
```

On the explorer, the Contract Registry can help you locate a contract by:

```text
name
creator
address
TXID
block
protocol
status
```

Then choose:

```text
INSPECT
```

to view the detailed contract record.

---

# 🧾 TRANSACTION RECEIPTS

TRU Core provides transaction receipts for contract operations.

A typical receipt may include:

```text
STATUS
TYPE
TXID
CONTRACT
AMOUNT
FEE
SENDER
FAMILY
STABLE ROOT
LIVE ANCHOR
```

The most important line is the status.

If it says:

```text
MEMPOOL ACCEPTED
```

wait for confirmation before treating the resulting contract state as final.

---

# 🧰 WHICH CONTRACT SHOULD I USE?

If you want...

### “Do not let this TRU move until later.”

Use:

```text
TIME LOCK
```

### “Only someone who knows this secret can claim it.”

Use:

```text
HASH LOCK
```

### “Release or execute based on a deterministic data feed.”

Use:

```text
ORACLE LOCK
```

### “Store something that can change over time.”

Use:

```text
STATEFUL KEY / VALUE
```

### “Let people vote and keep the tally on-chain.”

Use:

```text
VOTING
```

### “Create a rule-driven token issuer.”

Use:

```text
TOKEN ISSUER
```

### “Require two people out of three to approve.”

Use:

```text
MULTISIG / ESCROW
```

### “Swap assets without giving a middleman custody.”

Use:

```text
HTLC / ATOMIC SWAP
```

### “Put permanent data/reference material on-chain.”

Use:

```text
OP_RETURN
```

### “I am a developer and need an approved lower-level script.”

Use:

```text
CUSTOM SCRIPT
```

### “I want the specialized target/secret lock facility.”

Use:

```text
MAGICLOCK
```

---

# 🛡 SAFETY CHECKLIST

Before using a contract with meaningful value:

- **Start small.** Test with a small amount of TRU first.
- **Wait for confirmation.** Mempool acceptance is not confirmation.
- **Save the TXID/outpoint.**
- **Back up your wallet.**
- **Never share private keys or seed material.**
- **Protect preimages/secrets until you intend to reveal them.**
- **Verify recipients carefully before signing Multisig releases.**
- **Understand HTLC timeouts before funding them.**
- **Never put secrets in OP_RETURN.**
- **Do not use Custom Script unless you understand the script you are creating.**
- **Check the explorer after the transaction confirms.**
- **Keep Core/RPC access private and authenticated.**

---

# 🧑‍💻 ADVANCED: CONTRACT FAMILIES

Some TRU contracts have a canonical family identity.

Current stateful/application family names include:

```text
stateful_kv_v1
voting_v1
token_issuer_v1
multisig_escrow_2of3_v1
htlc_atomic_swap_v1
```

Other script-enforced contract types such as Time Lock, Hash Lock, Oracle Lock, OP_RETURN, Custom Script, and MagicLock do not all need to behave like the same stateful registry family.

That is intentional.

---

# 🗃 ADVANCED: CONTRACT REGISTRY MODEL

For stateful contracts, the conceptual registry is:

```text
creation outpoint
      │
      ▼
  Stable Root
      │
      ├──► Contract Family
      │
      └──► Current Live Anchor
                    │
                    ▼
              State Transition
                    │
                    ▼
              New Live Anchor
```

In storage terms, TRU uses relationships such as:

```text
contractlineage:<outpoint>   → <stable-root>
contractfamily:<stable-root> → <family>
contractlive:<stable-root>   → <current-live-anchor>
```

The purpose is simple:

> A state update should not accidentally create a brand-new logical contract identity.

---

# 🌎 CONTRACTS VS. TOKENS

A token and a contract are not automatically the same thing.

Example:

```text
NFT / FT / SFT / NCFT
        │
        ▼
   TOKEN REGISTRY
```

But:

```text
TOKEN ISSUER CONTRACT
        │
        ▼
   CONTRACT REGISTRY
        │
        ▼
may issue token units
```

The Token Issuer contract is the machine.

The issued token is the product coming out of the machine.

---

# 🔄 HTLC VS. BRIDGE

These are also different concepts.

### HTLC / Atomic Swap

```text
asset A remains on chain A
asset B remains on chain B
conditional contracts coordinate the exchange
```

### Bridge

A bridge usually introduces external-chain verification and/or representations of assets across chains.

This README documents the working **HTLC / Atomic Swap** contract capability.

It intentionally does not document retired/legacy bridge surfaces.

---

# 📚 SIMPLE END-TO-END EXAMPLE

Suppose you want to make a 1 TRU Time Lock.

```text
1. Start TRU Core
2. Let it fully synchronize
3. Open Create Contract
4. Choose Time Lock
5. Enter 1 TRU
6. Enter the requested maturity
7. Confirm
8. Save the TXID
9. Wait for a block
10. Open Contracts in the explorer
11. Search your TXID/address
12. Inspect the contract
13. Wait until it matures
14. Redeem it
15. Wait for another block
16. Verify the old lock is spent
17. Verify the new recipient output
```

That same mental model applies to most TRU contracts:

```text
BUILD IT
   ↓
BROADCAST IT
   ↓
CONFIRM IT
   ↓
USE IT
   ↓
CONFIRM AGAIN
   ↓
VERIFY IT
```

---

# 🧪 BUILDERS: WHAT A GOOD CONTRACT TEST SHOULD PROVE

A proper test should not merely report “PASS” because an error occurred.

For a contract with a positive and negative path, test both.

Example Time Lock:

```text
CONFIRM LOCK
     │
     ├── redeem too early → exact maturity rejection
     │
     └── redeem after maturity → successful spend
```

Example Hash Lock:

```text
CONFIRM LOCK
     │
     ├── wrong preimage   → rejected
     │
     └── correct preimage → successful spend
```

Example Multisig:

```text
CONFIRM ESCROW
     │
     ├── one signature    → insufficient
     │
     └── two valid keys   → release
```

Example HTLC:

```text
CONFIRM FUNDING
     │
     ├── correct claim path → successful claim
     │
     └── matured refund path → successful refund
```

The test should prove **why** something failed or succeeded.

---

# ❓ QUICK FAQ

## Is a TRU contract an Ethereum contract?

No.

TRU is its own blockchain and uses its own transaction/script/contract architecture.

## Do I need Ethereum gas?

No.

TRU transactions use TRU's own fee rules.

## Does “mempool accepted” mean confirmed?

No.

Wait until the transaction is mined into the active chain.

## Can a Time Lock be spent early?

A valid premature redemption should be rejected by the chain's maturity rules.

## Can anyone redeem a Hash Lock if they know the secret?

The exact authorization depends on the contract construction, but you should assume a revealed preimage is public. Protect it until intended use.

## Does Multisig send my private keys to the other signers?

No. Signers exchange signatures/public information, not private keys.

## Is an HTLC the same as a bridge?

No.

An HTLC is a conditional claim/refund primitive used for atomic swaps.

## Is OP_RETURN spendable?

No. It is intentionally a data output with zero spendable value.

## Is a Token Issuer itself a token?

No.

It is a contract that can issue token units under configured rules.

## Can I use Custom Script for anything I want?

No.

Custom scripts still have to satisfy TRU's recognized structural/relay/validation rules.

---

# 🧭 CURRENT PUBLIC CONTRACT SET

For a clean public-facing list, think of TRU's working contract stack as:

```text
TRU CONTRACTS
│
├── CONDITIONAL VALUE
│   ├── Time Lock
│   ├── Hash Lock
│   ├── Oracle Lock
│   ├── Multisig / Escrow
│   └── HTLC / Atomic Swap
│
├── STATEFUL APPLICATIONS
│   ├── Stateful Key / Value
│   ├── Voting
│   └── Token Issuer
│
├── DATA / DEVELOPER
│   ├── OP_RETURN
│   └── Custom Script
│
└── SPECIALIZED
    └── MagicLock
```

---

# 🚀 FINAL THOUGHT

The easiest way to understand TRU contracts is:

> **TRU lets you put rules around value and state, and the chain decides whether those rules were satisfied.**

Some contracts answer:

```text
"Has enough time passed?"
```

Some answer:

```text
"Do you know the secret?"
```

Some answer:

```text
"Did two of the three approved people sign?"
```

Some answer:

```text
"What is the current state?"
```

And an HTLC combines multiple rules so two parties can coordinate an exchange without handing custody to a central middleman.

Start with small amounts, wait for confirmations, inspect what happened on-chain, and build from there.

**Build it → Confirm it → Use it → Verify it.**
