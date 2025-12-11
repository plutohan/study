# Sui Move Comparison

# 1. Core Philosophy of Move Language
Move is not just a smart contract language but a **language designed to guarantee "Asset Safety" at the language level**.

The philosophy of Move can be summarized in three main points:

---

## 1.1 Linear Type System — "Assets cannot be copied/dropped"
Move's Resource type cannot be **copied or dropped**, meaning the following are prohibited:

- Copy → Causes Double Spend
- Drop → Causes Asset Loss
- Arbitrary Creation/Destruction → Causes Mint/Burn errors

Therefore, all assets (Resources) **must be moved via `move`**.

This is fundamentally different from generic number types in Solidity or Rust.

### Solidity Example
```solidity
uint balance = balances[msg.sender]; // Balance is a simple data structure, so it can be arbitrarily copied or deleted
```

### Move Example (Safe Structure)
```move
let c2 = c1;    // copy impossible
let c2 = move(c1);  // only move is possible
```

Move prevents Double Spend, Burn Bugs, etc., **at the compiler level from the source**.

---

## 1.2 Move Reinterprets Rust's Borrow Concept
Move inherits Rust's Ownership/Borrow philosophy, but the purpose is different:

| Rust Purpose | Move Purpose |
|----------|-----------|
| Memory Safety | Asset Safety |

Move also has `&T`, `&mut T` borrows like Rust.

Features of Move borrow rules:
- Assets cannot be moved while borrowed
- Only one mutable borrow allowed
- Multiple immutable borrows allowed
- Assets can only be moved/used after the borrow ends

Thanks to these rules, Sui ensures high stability even when handling the internal state of Objects.

```move
let slot = vector::borrow_mut(&mut lottery.slots, index);
```

If Rust's borrow-checker protects memory,
**Move's borrow-checker protects assets**.

---

## 1.3 Module + Ability System — Permission and Asset Security Model
In Move, the authority to create/modify a specific asset (Resource) **exists only within that module**.

Example:

```move
module mycoin {
    struct Coin has key {}  // no copy/drop
}
```

The function to mint/burn this Coin exists only inside this module.
It is linguistically impossible for external modules to arbitrarily delete or manipulate the Coin.

Also, the Ability system defines the security properties of a Resource:

- `key`: Can be used as an on-chain object
- `store`: Can be stored inside other structs
- `copy`: Copiable
- `drop`: Droppable

Assets (Resources) generally lack copy/drop → Copy/Deletion prohibited.

---

# 2. Unique Features of Sui Move — Object-Based Blockchain

Another innovation added by Sui on top of Move is the **Object Model**.

## 2.1 Object-Centric Model
Sui manages all on-chain assets as "Objects".

Objects include the following information:

- Object ID
- Owner
- Version
- Contents (Move struct)

### Advantages
- Independent Objects execute **automatically in parallel**
- Changes to each Object are independent → maximizes parallel processing performance

---

## 2.2 Powerful Sui CLI
Sui CLI is built-in.

- Object query (`sui client object <id>`)
- Event query
- Transaction preview
- Local/Testnet/Mainnet switchover
- Consistent Develop-Test-Deploy flow

The completeness is high enough that developers can do most on-chain development "without additional tools".

---

## 2.3 Powerful Public Node / RPC Fetch Model

Sui RPC supports **Object Fetch natively**.

Example:
```
sui client object 0x123...
```

With this one line, you can query:
- All fields of the Object
- Owner information
- Version
- Dynamic Fields
- Shared Object state

### You can build apps sufficiently without indexing services (e.g., Graph, Helius)

---

# 3. Comparison with Solidity/Solana using actual Lottery code

## defined in Move
```move
public struct Lottery has key {
  id: UID,
  creator: address,
  slots: vector<bool>,
  winner: option::Option<address>,
  prize: Balance<SUI>,
  remaining_fee: Balance<SUI>,
  fee: u64
}
```
- `has key` → Object stored on-chain
- Independent per Object → Parallel execution possible

---

### If defined in Solidity
```solidity
contract Lottery {
    address creator;
    bool[9] slots;
    address winner;
    uint256 fee;
    uint256 prize;
}
```

- All data is in a single global Storage
- To make multiple Lotteries, usually use mapping
- Parallel execution impossible

---

### If Solana (Anchor)
```rust
#[account]
pub struct Lottery {
    pub creator: Pubkey,
    pub slots: [bool; 9],
    pub winner: Option<Pubkey>,
    pub fee: u64,
}
```

- Need to calculate PDA/Seed to find the corresponding Account address Offchain
- Need to calculate size and rent
- Parallel processing possible

---

## Asset and Fee Handling

### Sui Move
```move
assert!(coin::value(&payment) == lottery.fee, EInsufficientPayment);
balance::join(&mut lottery.remaining_fee, coin::into_balance(payment));
```

- `Coin<SUI>` is a real asset Object and is placed under the Lottery Object
- Linear Type → Copy/Delete prohibited
- Asset bugs are blocked linguistically

### Solidity
```solidity
require(msg.value == fee);
remainingFee += msg.value;
```

- remainingFee is a simple uint256
- No asset meaning
- Compiler cannot know if mistakes happen

### Solana
```rust
pub fn pick_slot(ctx: Context<PickSlot>) -> Result<()> {
    let payer = &ctx.accounts.payer;
    let lottery = &mut ctx.accounts.lottery;

    // lamports payment (move amount equal to fee)
    let fee = lottery.fee;

    **payer.to_account_info().try_borrow_mut_lamports()? -= fee;
    **lottery.to_account_info().try_borrow_mut_lamports()? += fee;

    lottery.remaining_fee += fee;

    Ok(())
}
```
- Like Solidity, mistakes can occur in fee calculation

---

## Random and Probability Calculation

### Sui Move
```move
let mut generator = new_generator(r, ctx);
let random_number = generator.generate_u64_in_range(0, 10000);
```

- Protocol level random number
- No need for Chainlink
- Suitable for on-chain games

### Solidity, Solana
- Need external VRF like Chainlink/Switchboard/Pyth

---

## Capability-Based Access Control

### Sui Move
```move
let cap = allowlist::create_allowlist(name, ctx);
allowlist::transfer_cap(cap, sender);
```

- Permission itself is put into an Object called Cap and transferred to the admin address
- In other words, permission itself is an asset
- Therefore, query, transfer, and revocation of permission are very intuitive

## Solidity/Solana
- Permissions are hidden inside contract state


---
