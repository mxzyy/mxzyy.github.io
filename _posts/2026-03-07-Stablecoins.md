---
title: "Stablecoins"
date: 2026-03-07 08:30 +0700
categories: [Blockchain, Security]
tags: [stablecoins, defi, makerdao, dai, usdc, usdt, terra-luna, algorithmic-stablecoins]
author: mxzyy
---

A stablecoin is a crypto token designed to maintain relative stability of purchasing power. That definition matters. Calling stablecoins "digital dollars" misses the point—some are pegged to the US dollar, others to gold, and some float entirely. The core idea is **stability relative to some reference**, not a fixed parity to fiat currency. The mechanisms used to achieve that stability—and the tradeoffs they introduce—are what separate functional designs from catastrophic ones.

---

## The Classification Framework

Patrick Collins and Cyfrin introduced a classification framework that maps any stablecoin across three independent dimensions. This framework is useful because it exposes the structural risk profile of a stablecoin before you even look at its implementation.

### Dimension 1: Relative Stability

How the token maintains its target value.

- **Anchored (Pegged):** The token targets a fixed exchange rate against a reference asset. USDC targets $1.00. DAI targets $1.00. The mechanism to maintain that peg varies, but the intent is a hard anchor.
- **Floating:** The token does not target a specific price. Instead, it aims to preserve purchasing power over time—similar to how a central bank might target inflation rather than an exchange rate. RAI is the canonical example: it has a redemption price that floats based on market dynamics.

### Dimension 2: Stability Mechanism

How the peg (or stability target) is enforced.

- **Governed:** A centralized entity or governance body actively manages the stability. Tether Limited decides how USDT reserves are managed. Circle decides USDC's reserve composition. When something breaks, humans intervene.
- **Algorithmic:** Smart contract logic automatically adjusts supply, demand, or incentives to maintain stability. No human intervention required in the normal case. MakerDAO's liquidation system is algorithmic. Terra's mint/burn mechanism was algorithmic.

### Dimension 3: Collateral Type

What backs the stablecoin's value.

- **Exogenous:** Collateral comes from outside the stablecoin's own ecosystem. USDC is backed by US Treasuries and cash—assets whose value is completely independent of USDC's existence. DAI is backed by ETH, USDC, and other tokens that exist regardless of whether DAI exists.
- **Endogenous:** Collateral comes from within the stablecoin's own ecosystem. UST was backed by LUNA, a token whose primary value proposition was... being the collateral for UST. This creates a circular dependency.

---

## Mapping Real Stablecoins to the Framework

| Stablecoin | Relative Stability | Stability Mechanism | Collateral Type | Notes |
|---|---|---|---|---|
| **USDT** | Anchored | Governed | Exogenous | Tether Limited manages reserves (Treasuries, cash, commercial paper) |
| **USDC** | Anchored | Governed | Exogenous | Circle manages reserves (Treasuries, cash). Regulated, audited |
| **DAI** | Anchored | Algorithmic | Exogenous | Over-collateralized by ETH, USDC, etc. Smart contract liquidations |
| **UST** | Anchored | Algorithmic | Endogenous | Backed by LUNA. Collapsed May 2022 |
| **RAI** | Floating | Algorithmic | Exogenous | Backed by ETH. No hard peg—redemption price floats |
| **AMPL** | Floating | Algorithmic | Endogenous | Rebase mechanism. No collateral—supply adjusts directly |
| **FRAX** | Anchored | Algorithmic | Hybrid | Partially exogenous (USDC), partially endogenous (FXS) |

The critical pattern: **Anchored + Algorithmic + Endogenous is the most dangerous combination**. More on this below.

---

## Key DeFi Events

### UST/Luna Death Spiral — May 2022

Terra's UST maintained its peg through an arbitrage mechanism with LUNA:

- **UST above peg:** Users mint 1 UST by burning $1 worth of LUNA. Sell UST at market price > $1.00 for profit. UST supply increases, price drops toward peg.
- **UST below peg:** Users burn 1 UST to mint $1 worth of LUNA. Buy UST at market price < $1.00, burn it, receive LUNA worth more. UST supply decreases, price rises toward peg.

The problem is the circular dependency. When confidence breaks:

```
1. UST drops below peg
2. Arbitrageurs burn UST → mint LUNA
3. LUNA supply inflates → LUNA price drops
4. LUNA backs UST → UST's "collateral" is now worth less
5. More UST sells → more UST below peg
6. Go to step 2
```

This is a reflexive death spiral. The collateral's value depends on the stablecoin's stability, and the stablecoin's stability depends on the collateral's value. When the loop reverses, there is no floor.

On May 9, 2022, a large UST withdrawal from Anchor Protocol (~$2B) triggered the de-peg. Over the following days:

- UST fell from $1.00 to $0.035
- LUNA went from ~$80 to effectively $0 (inflated from 350M supply to 6.5T)
- ~$40B in combined market cap was destroyed

There was no external collateral to absorb the shock. The system only had its own token—and that token was in freefall.

### Black Thursday — March 12, 2020

On March 12, 2020, ETH dropped ~43% in a single day as COVID panic hit global markets. This triggered a mass liquidation event in MakerDAO.

The problem wasn't the liquidations themselves—that's the system working as designed. The problem was **gas prices**. Ethereum gas spiked so high that MakerDAO's keeper bots (the permissionless liquidation bots) couldn't submit transactions. Some keepers ran out of ETH for gas. Others had transactions stuck in the mempool.

The result: liquidation auctions completed with **zero bids**. Liquidators won ETH collateral for 0 DAI. MakerDAO accumulated ~$5.4M in bad debt because collateral was given away for free.

```
Normal liquidation:
  Vault under-collateralized → Keeper bids DAI → Wins collateral at discount → System solvent

Black Thursday:
  Vault under-collateralized → Gas too high → No keeper bids → Collateral sold for 0 DAI → Bad debt
```

This exposed a critical assumption in MakerDAO's design: the liquidation system assumed that keepers would always be able to participate in auctions. When network conditions prevented participation, the mechanism failed silently.

MakerDAO governance voted to mint and auction MKR tokens to recapitalize the ~$5.4M deficit.

### USDC De-peg — March 2023

On March 10, 2023, Silicon Valley Bank (SVB) collapsed. Circle had ~$3.3B of USDC reserves deposited at SVB. The market reacted immediately:

- USDC dropped to ~$0.87 on some exchanges
- DAI, which was ~50% backed by USDC via the PSM, also de-pegged to ~$0.90
- Curve's 3pool (USDC/USDT/DAI) became severely imbalanced as holders dumped USDC

The peg restored on March 13 when the FDIC guaranteed all SVB deposits. But the event exposed a fundamental tension: USDC is "safe" because it holds regulated reserves, but those reserves sit in a banking system that can fail over a weekend.

For DAI, the event demonstrated the cost of the PSM's tradeoff—by making DAI highly dependent on USDC for peg stability, MakerDAO imported USDC's centralization risk directly into DAI.

---

## MakerDAO Deep Dive

MakerDAO is the protocol behind DAI, the largest decentralized stablecoin. Understanding its mechanics is essential because it represents the most battle-tested approach to algorithmic, exogenously-collateralized stablecoin design.

### Vault System

Users deposit collateral (ETH, WBTC, USDC, etc.) into a Maker Vault and mint DAI against it. Every vault must maintain a minimum **collateralization ratio**.

```
Example (ETH-A vault, 150% min ratio):

  Deposit: 10 ETH @ $2,000 = $20,000 collateral
  Max DAI mintable: $20,000 / 1.50 = 13,333 DAI
  User mints: 10,000 DAI (200% ratio — safe buffer)

  If ETH drops to $1,500:
    Collateral value: 10 × $1,500 = $15,000
    Ratio: $15,000 / $10,000 = 150% → at liquidation threshold
```

When a vault's ratio falls below the minimum, it becomes eligible for liquidation. The system seizes the collateral and auctions it off to repay the DAI debt.

### Keepers

Keepers are permissionless bots that monitor vaults and trigger liquidations. Anyone can run a keeper—there's no whitelist, no permission, no registration. The incentive is profit: keepers buy auctioned collateral at a discount.

```
Keeper economics:
  Vault collateral: 10 ETH (worth $15,000)
  Vault debt: 10,000 DAI
  Liquidation penalty: 13% → total debt = 11,300 DAI

  Keeper bids 11,300 DAI → receives 10 ETH ($15,000)
  Profit: $15,000 - $11,300 = $3,700 (24.6% return)
```

Keepers serve a critical function: they keep the system solvent by ensuring under-collateralized positions are closed before they can create bad debt. The system relies on economic incentives rather than trusted operators.

### Liquidations 1.0 vs 2.0

**Liquidations 1.0 (Original):**

English auction format. Keepers bid increasing amounts of DAI for fixed collateral. Auctions had a fixed duration. This is what failed on Black Thursday—when no one could bid due to gas costs, collateral went for 0 DAI.

Problems:
- Capital inefficient (keepers needed DAI ready before auction)
- Slow (fixed auction duration)
- Vulnerable to network congestion (as Black Thursday proved)

**Liquidations 2.0 (Dutch Auction — post April 2021):**

The auction starts at a high price and decreases over time. The first keeper to accept the price wins.

```
Dutch auction price curve:

Price
  │
  │ ████
  │     ████
  │         ████
  │             ████
  │                 ████  ← Keeper buys here (acceptable discount)
  │                     ████
  │                         ████
  └──────────────────────────────── Time
```

Advantages:
- Instant settlement (no waiting for auction to end)
- Flash loan compatible (keepers don't need pre-existing capital)
- Price discovery happens naturally (first acceptable price wins)
- Faster collateral recycling

With flash loans, a keeper can:
1. See a vault eligible for liquidation
2. Take a flash loan for DAI
3. Bid on the auction
4. Receive collateral
5. Sell collateral for DAI
6. Repay flash loan
7. Keep profit

All in a single transaction. This dramatically increases the number of potential liquidators and reduces the risk of zero-bid scenarios.

### Peg Stability Module (PSM)

The PSM allows users to swap USDC for DAI at a fixed 1:1 rate (minus a small fee). This creates a hard floor and ceiling for DAI's price.

```
DAI above $1.00:
  User deposits 1 USDC into PSM → receives 1 DAI
  Sells DAI at market price > $1.00
  Arbitrage profit pushes DAI back to $1.00

DAI below $1.00:
  User buys 1 DAI on market for < $1.00
  Redeems 1 DAI at PSM → receives 1 USDC
  Arbitrage profit pushes DAI back to $1.00
```

The PSM is extremely effective at maintaining the peg. But it comes with a significant tradeoff: DAI becomes directly backed by USDC. At various points, USDC has comprised over 50% of DAI's collateral. This means:

- If USDC de-pegs (as in March 2023), DAI de-pegs
- If Circle blacklists the PSM contract, DAI loses its peg mechanism
- DAI inherits USDC's regulatory and counterparty risk

This is the central tension in MakerDAO's design: the PSM provides reliable peg stability, but at the cost of decentralization. MakerDAO governance has been gradually reducing USDC exposure, but removing it entirely would weaken the peg mechanism.

---

## Mathematics of Overcollateralized Stablecoins

The MakerDAO mechanics above—vaults, keepers, liquidations, the PSM—are all implementations of underlying mathematical relationships. This section formalizes those relationships. Every formula here maps directly to on-chain logic: collateralization checks in `Vat.frob()`, liquidation triggers in `Dog.bark()`, interest accumulation in `Jug.drip()`. Understanding the math means understanding exactly when and why a position becomes dangerous.

### 1. Collateralization Ratio (CR)

The collateralization ratio measures how much collateral value backs each unit of debt. It is the single most important number for any vault position.

$$
CR = \frac{C \times P}{D}
$$

| Variable | Definition |
|---|---|
| `C` | Amount of collateral deposited (e.g., ETH quantity) |
| `P` | Current market price of collateral asset (e.g., USD per ETH) |
| `D` | Outstanding debt in stablecoins (e.g., DAI owed) |

**Worked example:**

```
Given:
  C = 10 ETH
  P = $2,000 per ETH
  D = 10,000 DAI

CR = (10 × $2,000) / $10,000
CR = $20,000 / $10,000
CR = 2.0 = 200%
```

A 200% CR means the vault holds $2 of collateral for every $1 of debt. This is a healthy position with significant buffer.

**Edge case — approaching liquidation:**

```
ETH drops to $1,500:
  CR = (10 × $1,500) / $10,000
  CR = $15,000 / $10,000
  CR = 1.5 = 150%

For an ETH-A vault (150% minimum), this position is now at the liquidation boundary.
Any further price drop triggers liquidation.
```

**Relationship to other formulas:** CR is the foundation. The health factor (§2) normalizes CR against the liquidation threshold. Maximum mintable debt (§3) is derived by solving CR for `D`. The liquidation price (§4) is derived by solving CR for `P` at the minimum threshold.

---

### 2. Health Factor (Hf)

The health factor expresses how close a position is to liquidation as a single number. When `Hf < 1`, the position is liquidatable.

$$
H_f = \frac{C \times P \times L_t}{D}
$$

| Variable | Definition |
|---|---|
| `C` | Amount of collateral deposited |
| `P` | Current market price of collateral |
| `L_t` | Liquidation threshold (expressed as a decimal, e.g., 0.6667 for a 150% min CR) |
| `D` | Outstanding debt |

The liquidation threshold `L_t` is the inverse of the minimum collateralization ratio: `L_t = 1 / CR_min`. For a 150% minimum CR, `L_t = 1/1.5 ≈ 0.6667`. For a 200% minimum CR (like ETH-C vaults in MakerDAO), `L_t = 1/2.0 = 0.5`.

**Why 50% (200% overcollateralization) is a common threshold:**

A 50% liquidation threshold (`L_t = 0.5`, i.e., `CR_min = 200%`) means collateral can lose half its value before the position becomes under-collateralized. This buffer accounts for:
- Volatile assets (ETH can drop 30%+ in a day, as Black Thursday showed)
- Liquidation execution delay (time between price drop and actual auction completion)
- Liquidation penalty overhead (the penalty itself consumes collateral)
- Oracle latency (price feeds update periodically, not continuously)

Lower-volatility collateral types get tighter thresholds (e.g., USDC vaults at 101%), while volatile assets need wider buffers.

**Worked example — safe position:**

```
Given:
  C = 10 ETH
  P = $2,000
  L_t = 0.6667 (150% min CR, ETH-A vault)
  D = 10,000 DAI

Hf = (10 × $2,000 × 0.6667) / $10,000
Hf = $13,334 / $10,000
Hf = 1.33

Hf > 1 → Position is safe. Collateral can still drop ~25% before liquidation.
```

**Edge case — liquidation imminent:**

```
ETH drops to $1,400:
  Hf = (10 × $1,400 × 0.6667) / $10,000
  Hf = $9,334 / $10,000
  Hf = 0.93

Hf < 1 → Position is liquidatable. Keepers can trigger liquidation immediately.
```

**Relationship to other formulas:** Health factor is a normalized form of CR. When `Hf = 1`, `CR = CR_min`—the position is exactly at the liquidation boundary. The liquidation price (§4) is the price `P` that makes `Hf = 1`. The liquidation penalty (§5) determines how much collateral is actually seized once `Hf` drops below 1.

---

### 3. Maximum Mintable (Dmax)

Given a collateral deposit, the maximum stablecoin debt a user can take on before hitting the minimum collateralization ratio.

$$
D_{max} = \frac{C \times P}{CR_{min}}
$$

| Variable | Definition |
|---|---|
| `C` | Amount of collateral deposited |
| `P` | Current market price of collateral |
| `CR_{min}` | Minimum collateralization ratio (e.g., 1.5 for 150%) |

**Worked example:**

```
Given:
  C = 10 ETH
  P = $2,000
  CR_min = 1.5 (ETH-A vault)

Dmax = (10 × $2,000) / 1.5
Dmax = $20,000 / 1.5
Dmax = 13,333.33 DAI

This is the absolute maximum. Minting this amount puts the vault at exactly
150% CR — one wei of price movement triggers liquidation.
```

In practice, users should mint well below `Dmax`. A common heuristic is to target a CR of 200-300%, which means minting only 50-75% of `Dmax`:

```
Safe mint (targeting 250% CR):
  D_safe = (10 × $2,000) / 2.5
  D_safe = 8,000 DAI

This leaves room for a 40% ETH price drop before liquidation:
  CR at $1,200 ETH: (10 × $1,200) / $8,000 = 150% → just at boundary
```

**Edge case — minting at maximum:**

```
User mints Dmax = 13,333 DAI (exactly 150% CR)

ETH drops 1% to $1,980:
  CR = (10 × $1,980) / $13,333 = 148.5%
  → Below 150% minimum. Liquidation triggered.

The user loses 13% liquidation penalty on top of the price drop.
Total loss: collateral seized, penalty applied, remaining collateral returned
(if any). Minting at Dmax is effectively gambling that the collateral price
will never move down.
```

**Relationship to other formulas:** `Dmax` is CR (§1) solved for `D` at the minimum threshold. Minting `Dmax` sets `Hf = 1` (§2). The stability fee (§6) means actual debt grows over time, so even a position minted below `Dmax` can approach the liquidation boundary as interest accrues.

---

### 4. Liquidation Price

The collateral price at which a position becomes liquidatable. Derived by setting `Hf = 1` and solving for `P`.

$$
P_{liq} = \frac{D \times CR_{min}}{C}
$$

| Variable | Definition |
|---|---|
| `D` | Outstanding debt |
| `CR_{min}` | Minimum collateralization ratio |
| `C` | Amount of collateral deposited |

**Derivation:** Start with the CR equation at the liquidation boundary:

```
CR_min = (C × P_liq) / D
→ P_liq = (D × CR_min) / C
```

**Worked example:**

```
Given:
  C = 10 ETH
  D = 10,000 DAI
  CR_min = 1.5

P_liq = ($10,000 × 1.5) / 10
P_liq = $15,000 / 10
P_liq = $1,500

If ETH falls to $1,500, this vault gets liquidated.
Current ETH price is $2,000, so there's a $500 (25%) buffer.
```

**Edge case — high leverage:**

```
Same collateral, but user minted 12,000 DAI instead:

P_liq = ($12,000 × 1.5) / 10
P_liq = $1,800

Now liquidation triggers at $1,800 — only a 10% drop from $2,000.
A single volatile day (like Black Thursday's 43% drop) would blow
through this level instantly.
```

**Edge case — stability fee erosion:**

```
User opens vault with D = 10,000 DAI. P_liq = $1,500 initially.
After 2 years at 5% stability fee:

D_actual = 10,000 × (1.05)^2 = 11,025 DAI
P_liq = ($11,025 × 1.5) / 10 = $1,653.75

The liquidation price crept up by $153.75 — without the user doing anything.
Stability fees silently erode the safety buffer.
```

**Relationship to other formulas:** Liquidation price is the inverse of maximum mintable (§3)—one solves for `D`, the other for `P`, both at the CR boundary. The stability fee (§6) continuously increases `D`, which continuously increases `P_liq`. Once liquidation triggers, the penalty formula (§5) determines the actual cost.

---

### 5. Liquidation Penalty & Liquidator Profit

When a vault is liquidated, the system doesn't just seize enough collateral to cover the debt—it adds a penalty. This penalty incentivizes keepers to perform liquidations promptly and creates a cost for users who let their positions become unhealthy.

**Collateral seized:**

$$
C_{seized} = \frac{D \times (1 + \lambda)}{P}
$$

| Variable | Definition |
|---|---|
| `D` | Outstanding debt at time of liquidation |
| `λ` (lambda) | Liquidation penalty rate (e.g., 0.13 for MakerDAO's 13% on ETH-A) |
| `P` | Collateral price at time of liquidation |

**Liquidator profit:**

$$
\pi = C_{seized} \times P - D = D \times \lambda
$$

In a competitive market, the liquidator's gross profit approaches `D × λ` (the penalty amount), minus gas costs and any discount from the Dutch auction price.

**Worked example:**

```
Given:
  C = 10 ETH
  D = 10,000 DAI
  P = $1,450 (just below liquidation price of $1,500)
  λ = 0.13 (13% penalty)

Collateral seized:
  C_seized = $10,000 × (1 + 0.13) / $1,450
  C_seized = $11,300 / $1,450
  C_seized = 7.79 ETH

Value of seized collateral: 7.79 × $1,450 = $11,300
Liquidator pays: 10,000 DAI
Liquidator gross profit: $11,300 - $10,000 = $1,300 (= D × λ)

Vault owner receives remaining collateral:
  10 - 7.79 = 2.21 ETH ($3,204.50)

Vault owner's total loss:
  Original: 10 ETH ($14,500) - 10,000 DAI debt = $4,500 net equity
  After liquidation: 2.21 ETH ($3,204.50) - 0 DAI debt = $3,204.50
  Loss from liquidation: $4,500 - $3,204.50 = $1,295.50 (≈ the 13% penalty)
```

**Edge case — collateral insufficient to cover debt + penalty:**

```
ETH crashes to $1,050 before liquidation executes (oracle delay, gas spike):

  C_seized = $10,000 × 1.13 / $1,050 = 10.76 ETH

But the vault only has 10 ETH. The system can seize at most 10 ETH:
  Value: 10 × $1,050 = $10,500
  Debt: $10,000
  Penalty coverage: only $500 of the $1,300 penalty is covered

  Remaining: $10,500 - $10,000 = $500 for the penalty
  Bad debt: $0 (debt is covered, but penalty is partially absorbed)

If ETH drops further to $900:
  Collateral value: 10 × $900 = $9,000
  Debt: $10,000
  Bad debt: $1,000 — the system cannot fully recover the loan.
  This is how MakerDAO accumulated $5.4M in bad debt on Black Thursday.
```

**Relationship to other formulas:** The penalty `λ` is what makes liquidation profitable for keepers, which is what makes the CR (§1) and Hf (§2) thresholds enforceable. Without profitable liquidations, minimum CR requirements are unenforceable promises. The penalty also means the effective cost of liquidation exceeds what the simple liquidation price (§4) suggests—users lose more than just the price difference.

---

### 6. Stability Fee (Compound Interest)

The stability fee is MakerDAO's version of an interest rate. It's charged on outstanding DAI debt and accrues continuously. The fee is set per-collateral-type by MKR governance and serves two purposes: generating protocol revenue and acting as a monetary policy lever (higher fees discourage borrowing, reducing DAI supply).

$$
D(t) = D_0 \times \left(1 + r\right)^t
$$

For continuous compounding (closer to on-chain implementation):

$$
D(t) = D_0 \times e^{r \cdot t}
$$

| Variable | Definition |
|---|---|
| `D_0` | Initial debt (DAI minted) |
| `r` | Annual stability fee rate (e.g., 0.05 for 5%) |
| `t` | Time in years |
| `D(t)` | Total debt owed at time `t` |

On-chain, MakerDAO uses a per-second compounding rate. The `Jug` contract stores a rate value that is multiplied into the cumulative rate accumulator on every `drip()` call. The per-second rate for 5% annual is:

```
r_per_second = (1.05)^(1/31536000) ≈ 1.0000000015854896
```

**Worked example (discrete annual compounding):**

```
Given:
  D_0 = 10,000 DAI
  r = 5% annually

After 1 year:
  D(1) = 10,000 × (1.05)^1 = 10,500 DAI
  Interest owed: 500 DAI

After 3 years:
  D(3) = 10,000 × (1.05)^3 = 11,576.25 DAI
  Interest owed: 1,576.25 DAI

After 5 years:
  D(5) = 10,000 × (1.05)^5 = 12,762.82 DAI
  Interest owed: 2,762.82 DAI
```

**Edge case — stability fee triggers liquidation:**

```
Initial position:
  C = 10 ETH, P = $2,000 (constant — no price movement)
  D_0 = 12,000 DAI
  CR_min = 1.5, r = 8% annual

Initial CR: (10 × $2,000) / $12,000 = 166.7% — above 150%, safe.

After 1 year:
  D(1) = 12,000 × 1.08 = 12,960 DAI
  CR = $20,000 / $12,960 = 154.3% — still safe, but shrinking.

After 2 years:
  D(2) = 12,000 × (1.08)^2 = 13,996.80 DAI
  CR = $20,000 / $13,996.80 = 142.9% — below 150%. Liquidated.

ETH price never moved. The stability fee alone pushed the position
below the liquidation threshold. Users must either:
  1. Pay down accrued fees periodically
  2. Add more collateral
  3. Maintain a large enough buffer to absorb fee growth
```

**Relationship to other formulas:** The stability fee continuously increases `D`, which decreases CR (§1), decreases Hf (§2), increases the liquidation price (§4), and means `Dmax` (§3) must be recalculated against the growing debt, not the original mint amount. It is the silent force that erodes every safety buffer over time.

---

### 7. Peg Arbitrage Profit

When DAI deviates from its $1.00 peg, arbitrageurs can profit by trading against the deviation, pushing the price back toward peg. The PSM makes this especially efficient by providing a guaranteed 1:1 swap.

**DAI below peg (buy DAI, redeem at PSM):**

$$
\pi = Q \times (1 - P_{DAI}) - G
$$

**DAI above peg (mint DAI via PSM, sell on market):**

$$
\pi = Q \times (P_{DAI} - 1) - G
$$

| Variable | Definition |
|---|---|
| `Q` | Quantity of DAI traded |
| `P_{DAI}` | Current market price of DAI (e.g., $0.98 or $1.03) |
| `G` | Total gas cost of the transaction(s) in USD |
| `π` | Arbitrage profit |

**Worked example — DAI below peg:**

```
Given:
  P_DAI = $0.98 (DAI trading at 2% discount)
  Q = 100,000 DAI
  G = $15 (gas for swap on DEX + PSM redemption)

Step 1: Buy 100,000 DAI on market
  Cost: 100,000 × $0.98 = $98,000

Step 2: Redeem 100,000 DAI at PSM for 100,000 USDC
  Received: $100,000

Profit:
  π = 100,000 × (1 - 0.98) - $15
  π = $2,000 - $15
  π = $1,985
```

**Worked example — DAI above peg:**

```
Given:
  P_DAI = $1.015
  Q = 50,000 DAI
  G = $15

Step 1: Deposit 50,000 USDC into PSM, receive 50,000 DAI
  Cost: $50,000

Step 2: Sell 50,000 DAI on market at $1.015
  Received: $50,750

Profit:
  π = 50,000 × (1.015 - 1) - $15
  π = $750 - $15
  π = $735
```

**Edge case — gas costs kill small arbitrage:**

```
P_DAI = $0.998 (0.2% deviation — small but real)
G = $30 (moderate gas)

Minimum profitable trade size:
  π > 0 → Q × (1 - 0.998) > $30
  Q × 0.002 > $30
  Q > 15,000 DAI

Any trade below 15,000 DAI loses money to gas.
At $0.999 deviation (0.1%), minimum Q > 30,000 DAI.

This is why small deviations can persist — they're not profitable
to arbitrage at retail scale. Large market makers with optimized
gas costs and MEV strategies can profitably arbitrage tighter bands.
```

**Edge case — PSM fee:**

```
If the PSM charges a fee (e.g., 0.1% tin/tout):
  π = Q × (1 - P_DAI) - G - Q × f_PSM

At $0.98 DAI with 0.1% PSM fee:
  π = 100,000 × 0.02 - $15 - 100,000 × 0.001
  π = $2,000 - $15 - $100
  π = $1,885

The PSM fee slightly reduces profit but doesn't eliminate the
arbitrage at this deviation level. At smaller deviations, the
PSM fee becomes a larger fraction of potential profit.
```

**Relationship to other formulas:** Peg arbitrage is the market mechanism that enforces DAI's $1.00 target. Without profitable arbitrage, the peg would drift. The stability fee (§6) indirectly affects peg dynamics—higher fees reduce DAI supply (users repay debt to avoid fees), which pushes DAI price up. The PSM provides the hard anchor that makes arbitrage risk-free (no slippage, guaranteed rate), but requires USDC reserves, which connects back to the centralization tradeoffs discussed in the PSM section above.

---

### Formula Reference Table

| # | Formula | What It Answers | Key Variables | Danger Signal |
|---|---|---|---|---|
| 1 | `CR = (C × P) / D` | How collateralized is my position? | Collateral, Price, Debt | `CR < CR_min` |
| 2 | `Hf = (C × P × L_t) / D` | How close am I to liquidation? | + Liquidation threshold | `Hf < 1` |
| 3 | `Dmax = (C × P) / CR_min` | How much can I borrow? | Collateral, Price, Min ratio | Minting near `Dmax` |
| 4 | `P_liq = (D × CR_min) / C` | What price liquidates me? | Debt, Min ratio, Collateral | `P_liq` close to current `P` |
| 5 | `C_seized = D(1+λ) / P` | What do I lose in liquidation? | + Penalty rate | `C_seized > C` (bad debt) |
| 6 | `D(t) = D_0 × (1+r)^t` | How does my debt grow? | Initial debt, Rate, Time | `D(t)` pushing CR toward `CR_min` |
| 7 | `π = Q × |1 - P_DAI| - G` | Is peg arbitrage profitable? | Quantity, DAI price, Gas | `G > Q × deviation` (unprofitable) |

These formulas are interconnected: the stability fee (6) increases debt `D`, which decreases CR (1) and Hf (2), raises the liquidation price (4), and increases the collateral seized if liquidation occurs (5). Peg arbitrage (7) is the external market force that makes the whole system worth building—without a reliable peg, none of the other mechanics matter.

---

## Rebase Stablecoins — Ampleforth (AMPL)

Ampleforth takes a radically different approach. Instead of using collateral or governance to maintain stability, AMPL adjusts its own supply. Every wallet's balance changes automatically based on the protocol's target price.

### How Rebasing Works

AMPL targets a price of $1.06 (2019 CPI-adjusted dollar). Once per day, the protocol checks the volume-weighted average price (VWAP):

- **Price > target (e.g., $1.20):** Positive rebase. Supply increases. Every holder's balance increases proportionally.
- **Price < target (e.g., $0.80):** Negative rebase. Supply decreases. Every holder's balance decreases proportionally.
- **Price at target:** No rebase.

```
Example — positive rebase:

  You hold: 1,000 AMPL
  Total supply: 10,000,000 AMPL
  Your share: 0.01%
  AMPL price: $1.50 (above $1.06 target)

  Rebase expands supply by 10%:
  New total supply: 11,000,000 AMPL
  Your new balance: 1,100 AMPL (still 0.01% of supply)
  Target effect: more supply → selling pressure → price drops toward $1.06
```

```
Example — negative rebase:

  You hold: 1,000 AMPL
  AMPL price: $0.60

  Rebase contracts supply by 10%:
  Your new balance: 900 AMPL
  Target effect: less supply → scarcity → price rises toward $1.06
```

Your **percentage ownership** of total supply never changes. What changes is the number of tokens in your wallet. This means AMPL doesn't maintain price stability in the traditional sense—it maintains **market cap stability** relative to demand, distributed across a changing number of tokens.

The rebase mechanism is endogenous and algorithmic: there's no external collateral, and no governance body deciding the supply adjustment. The smart contract executes the rebase based on oracle price data.

### The Problem with Rebasing

Rebasing creates cognitive and practical challenges:

- **Wallet balances change without transactions.** Users see their balance drop and panic sell, amplifying the negative rebase.
- **DeFi composability is difficult.** Lending protocols, AMMs, and other contracts need special handling for tokens whose balances change autonomously.
- **Volatility isn't eliminated—it's transferred.** Instead of price volatility, you get supply volatility. Your portfolio value still fluctuates.

AMPL has never maintained stable purchasing power in practice. It oscillates between expansion and contraction cycles, making it more of a volatility asset than a stablecoin.

---

## Why Anchored + Algorithmic + Endogenous Will Likely Always Fail

The three-dimension framework reveals why the combination of **Anchored + Algorithmic + Endogenous** is structurally unsound.

**Anchored** means the system promises a fixed price. This creates a rigid target that the mechanism must defend under all conditions—including extreme stress.

**Algorithmic** means no human backstop. When the mechanism fails, there's no governance body that can inject capital, pause redemptions, or restructure. The code runs as written.

**Endogenous** means the collateral's value depends on the system's own health. When confidence breaks, the collateral and the stablecoin enter a reflexive death spiral.

Put together:

```
Stress event hits
  → Stablecoin de-pegs
    → Arbitrage mechanism mints more endogenous collateral token
      → Collateral token supply inflates
        → Collateral token price drops
          → Stablecoin's backing is now worth less
            → Stablecoin de-pegs further
              → Repeat until value = 0
```

The fundamental issue is that there is **no exogenous value floor**. In a system like MakerDAO, if DAI drops below peg, the ETH collateral still has independent value—people use ETH for gas, staking, DeFi, regardless of whether DAI exists. The liquidation system can always sell real assets to cover debt.

In an endogenous system, the "collateral" has no independent demand source. LUNA's primary utility was being burned to mint UST. When UST failed, LUNA had no remaining value proposition. There was nothing to liquidate into.

Every attempt at this combination has failed:

| Protocol | Token Pair | Collapse Date | Peak TVL |
|---|---|---|---|
| Terra | UST / LUNA | May 2022 | ~$18B |
| Basis Cash | BAC / BAS | Jan 2021 | ~$175M |
| Iron Finance | IRON / TITAN | Jun 2021 | ~$2B |
| Empty Set Dollar | ESD / DSU | Jan 2021 | ~$100M |

The pattern is consistent: the system works during expansion (when demand is growing and reflexivity is positive) and collapses during contraction (when demand shrinks and reflexivity turns negative). There is no mechanism to arrest the spiral because the only tool available—minting more of the endogenous token—actively makes the problem worse.

This doesn't mean all algorithmic stablecoins fail. DAI is algorithmic and has survived multiple crises. RAI is algorithmic and floating. The critical variable is **exogenous collateral**: assets whose value doesn't depend on the stablecoin's success. Remove that, and you're building on a circular foundation.
