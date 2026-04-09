# Skill: Forge Pool Lifecycle Testing

> Current verified Forge pool deployment and lifecycle validation boundary.
> Use this when touching live testnet deployment, bootstrap, or V2 add/remove validation.

---

## Domain

This skill preserves the stable parts of the current Forge pool lifecycle test flow:

- which scripts define the live deploy and lifecycle path
- the current V2 versus legacy split
- what has been validated already
- what should stay in code and manifests instead of becoming durable skill knowledge

Use it for:

- live testnet deployment work
- LP lifecycle validation work
- avoiding regressions while editing deploy or liquidity scripts

---

## Current Verified Flow

### Stage 1

Launcher and bootstrap root creation live in:

- [projects/chia-cfmm/contracts/deploy_forge_pool_testnet.py](../../projects/chia-cfmm/contracts/deploy_forge_pool_testnet.py)

### Stage 2

Bootstrap and reserve seeding live in:

- [projects/chia-cfmm/contracts/deploy_forge_pool_testnet.py](../../projects/chia-cfmm/contracts/deploy_forge_pool_testnet.py)
- [projects/chia-cfmm/contracts/bootstrap_bundle_state.py](../../projects/chia-cfmm/contracts/bootstrap_bundle_state.py)

### Lifecycle operations

Current V2 add and remove logic lives in:

- [projects/chia-cfmm/contracts/forge_add_liquidity_v2.py](../../projects/chia-cfmm/contracts/forge_add_liquidity_v2.py)
- [projects/chia-cfmm/contracts/forge_remove_liquidity_v2.py](../../projects/chia-cfmm/contracts/forge_remove_liquidity_v2.py)
- [projects/chia-cfmm/contracts/live_pool_lifecycle_testnet.py](../../projects/chia-cfmm/contracts/live_pool_lifecycle_testnet.py)

### Puzzle hash derivation

Current V2 puzzle and announcement helpers live in:

- [projects/chia-cfmm/contracts/compute_pool_v2_ph.py](../../projects/chia-cfmm/contracts/compute_pool_v2_ph.py)

---

## Current Architecture Boundary

The current validated direction is V2.

Important durable facts:

- V2 pool state is 4-field, not legacy 8-field
- V2 uses a single-curry pool formula for all states
- V2 uses a unified pool announcement message
- current live lifecycle work assumes V2 add/remove builders, not legacy pool math

Do not mix legacy deploy/bootstrap assumptions with V2 liquidity builders.

---

## What Was Verified

The durable outcome from the latest validation slice is:

- the proper-tail deploy path was unified onto the V2 pool format in code
- bootstrap artifact reconstruction now handles V2 state
- lifecycle harness logic now includes real LP CAT delta spend construction instead of placeholder preview-only logic
- live V2 add/remove validation is now confirmed on testnet for a proper-tail 2-asset pool: bootstrap, two deposits, and a partial LP withdrawal all landed on-chain with matching reserve and supply transitions
- the LP mint path required a raw XCH value-funding spend; reserve-fee style funding was rejected during live probes
- dry-run V2 deploy construction completed coherently

---

## Current Blockers

Current blockers are environmental rather than architectural:

- Sage may be synced locally while still lacking peers for transaction submission
- if the new LP CAT asset never lands in the wallet, lifecycle validation that depends on LP CAT preview spends cannot proceed truthfully
- live swap validation is still blocked by execution wiring: current Forge swap-engine quotes can price pools, but they do not yet carry a truthful pool-aware take/submit path for on-chain settlement

That means a failed live run should first be checked as an infrastructure problem before changing contract logic.

---

## What Not to Treat as Stable Skill Knowledge

Do not freeze these into long-lived mental models:

- specific launcher ids
- one-off manifest files
- debug payload files
- temporary dry-run artifacts
- transient Sage or mempool failures

Keep those in code, manifests, or quest notes instead.
