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
