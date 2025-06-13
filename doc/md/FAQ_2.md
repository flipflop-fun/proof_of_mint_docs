# FAQ of flipflop.plus II

## A. About (Continued)

### A15. Is the number of minting operations per Checkpoint fixed?
**No, it is not fixed.** The number of minting operations in each Checkpoint is determined by the **target minting amount (T)** and the **dynamic single minting amount (M)**, with the total minting amount constrained to not exceed the target. Below are the key mechanisms and case studies:

#### 1. Core Calculation Rules
- **Single Minting Amount**:
  $M = \frac{M_b}{d}$
  - $M_b$: Base minting amount (adjusted by Milestone decay).
  - $d$: Current FOMO coefficient.

- **Number of Minting Operations**:
  $n = \left\lfloor \frac{T}{M} \right\rfloor$
  (Rounded down to ensure total minting amount $n \cdot M \leq T$.)

- **Remaining Unminted Amount**:
  $\text{Remaining Amount} = T - n \cdot M$

#### 2. User Case Study
| Parameter              | Value            |
|------------------------|------------------|
| Target Minting Amount ($T$) | 13,000 tokens    |
| Base Minting Amount ($M_b$) | 1,000 tokens     |
| FOMO Coefficient ($d$)      | 1.33             |

- **Calculation Process**:
  1. Single minting amount: $M = 1,000 / 1.33 \approx 751.8797$
  2. Maximum operations: $n = \lfloor 13,000 / 751.8797 \rfloor = 17$
  3. Total minted: $17 \times 751.8797 \approx 12,781.95$
  4. Remaining amount: $13,000 - 12,781.95 = 218.05$

**Conclusion**:
- The Checkpoint has 17 minting operations (non-fixed).
- The remaining amount is carried over to subsequent Checkpoints or compensated via difficulty adjustments.

#### 3. Extended Case Studies
##### Case 1: Increased FOMO Coefficient (Suppressing Rapid Minting)
| Parameter              | Value            |
|------------------------|------------------|
| Target Minting Amount ($T$) | 13,000 tokens    |
| Base Minting Amount ($M_b$) | 1,000 tokens     |
| FOMO Coefficient ($d$)      | **1.5** (due to faster minting) |

- **Calculation**:
  1. $M = 1,000 / 1.5 \approx 666.6667$
  2. $n = \lfloor 13,000 / 666.6667 \rfloor = 19$
  3. Total minted: $19 \times 666.6667 \approx 12,666.67$
  4. Remaining amount: $13,000 - 12,666.67 = 333.33$

**Impact**:
- Minting operations increase from 17 to 19, but single minting amount decreases, leaving a larger remaining amount.
- Higher FOMO coefficient raises costs in subsequent Checkpoints.

##### Case 2: Stable FOMO Coefficient (Steady Minting)
| Parameter              | Value            |
|------------------------|------------------|
| Target Minting Amount ($T$) | 13,000 tokens    |
| Base Minting Amount ($M_b$) | 1,000 tokens     |
| FOMO Coefficient ($d$)      | **1.0** (on-target minting) |

- **Calculation**:
  1. $M = 1,000 / 1.0 = 1,000$
  2. $n = \lfloor 13,000 / 1,000 \rfloor = 13$
  3. Total minted: $13 \times 1,000 = 13,000$
  4. Remaining amount: $0$

**Impact**:
- On-target minting results in the fewest operations with no remaining amount, maintaining stable costs.

#### 4. Remaining Amount Handling Mechanism
- Excessively high FOMO coefficients may lead to significant unminted amounts, creating deflationary pressure.
- Unminted tokens are periodically burned to mitigate the side effects of high FOMO coefficients.

#### 5. Design Summary
| Feature                | Description                                                           |
|------------------------|-----------------------------------------------------------------------|
| **Dynamic Operations** | Number of operations is dynamically calculated based on $d$ and $M_b$. |
| **Hard Cap Constraint**| Total minted amount never exceeds $T$; remaining amounts are burned periodically. |
| **Anti-Bot Design**    | Bots accelerating minting trigger $d↑ \rightarrow M↓ \rightarrow n↑$, but profits are limited by $T$ and rising costs. |

#### Conclusion
The number of minting operations per Checkpoint is **not fixed** and is dynamically calculated via **target amount constraints** and **FOMO coefficient regulation**. This design ensures precise token distribution with deflationary control while minimizing manipulation through mathematical rules, making it the first mechanism to achieve "algorithmically fair issuance."

---

### A16. How does the Rebase mechanism for minting costs and market prices after reaching the Target Milestone prevent a death spiral?
flipflop.plus employs a dual **Rebase mechanism** of **minting cost anchoring** and **liquidity self-reinforcement** to break the downward spiral common in traditional tokens. Below are the core principles and anti-spiral design:

#### 1. Core Logic of the Rebase Mechanism
- **Unidirectional Minting Cost Increase**:
  Each Milestone reduces the **base minting amount** (e.g., by 25%) and the **FOMO coefficient** increases, ensuring the minting cost $p$ rises continuously:
  $p = \frac{P_0 \cdot d}{M_0 \cdot f^{(m-1)}}$
  - $d$ is non-decreasing, and $f < 1$, resulting in a **strictly monotonic increasing** cost curve.

- **Dynamic Market Price Anchoring**:
  - **Floor Price Support**: If the market price falls below the current minting cost, arbitrageurs **buy tokens** rather than mint, increasing demand and pushing prices up, slowing minting.
  - **Ceiling Constraint**: If the market price exceeds the minting cost, arbitrageurs **mint tokens** instead of buying, increasing supply and liquidity injections, stabilizing prices.

#### 2. Three Lines of Defense Against Death Spirals
##### Defense Line 1: Liquidity Moat
- **Token Vault Auto-Injection**:
  Every minting fee contributes to the liquidity pool, creating **cumulative buy-side depth**.
- **Fee Reinvestment**:
  100% of DEX trading fees are reinvested into the pool, enhancing resilience (similar to Bitcoin’s miner fees securing the network).

##### Defense Line 2: Cost-Price Game Equilibrium
| Market Behavior                  | Rebase Response                       | Economic Effect                     |
|----------------------------------|---------------------------------------|-------------------------------------|
| Price falls near minting cost    | Arbitrageurs buy tokens → demand rises | Price rebounds above cost line      |
| Price crashes far below cost     | Minting halts → supply stops          | Prevents further sell-off pressure  |
| Price surges above cost          | Minting increases → liquidity injection curbs bubbles | Maintains reasonable valuation range |

##### Defense Line 3: Anti-Dumping Incentives
- **Delayed Dumping Penalty**:
  Hoarders holding tokens into later Milestones face:
  - **Higher Minting Costs**: New entrants pay higher prices, reducing sell-off competition.
  - **Growing Liquidity Depth**: Large dumps are absorbed by deeper pools, increasing dumping costs.

#### 3. Comparison with Traditional Models
| Scenario                | No Rebase (e.g., pump.fun)             | **flipflop.plus Rebase Mechanism**     |
|-------------------------|----------------------------------------|----------------------------------------|
| **Price Decline**       | Panic selling → liquidity depletion → death spiral | Arbitrage buying + liquidity injection supports price |
| **Bot Mass Dumping**    | No cost anchor, price free-falls       | Dumping below cost triggers automatic buy equilibrium |
| **Long-Term Stagnation**| Liquidity exhaustion, token collapses | Minting pause + Token Vault reserves maintain baseline value |

#### Summary
The Rebase mechanism builds a three-tier anti-fragile system through **rigid cost escalation** and **adaptive liquidity injection**:
1. **Short-Term**: Arbitrage corrects price deviations quickly.
2. **Mid-Term**: Liquidity moat absorbs abnormal volatility.
3. **Long-Term**: Cost anchoring provides an intrinsic value floor.
This design encodes traditional financial **market-making** and **value storage** into on-chain protocols, mathematically eliminating death spirals without relying on human intervention.

---

### A17. Are there plans to support non-Solana chains in the future?
flipflop.plus is designed with **protocol portability** and plans a **multi-chain deployment** across 2024–2027. Below is the detailed roadmap:

#### 1. Confirmed Supported Blockchains
| Category             | Blockchains                                   | Technical Progress                            |
|----------------------|-----------------------------------------------|-----------------------------------------------|
| **EVM-Compatible Minting Chains** | Solana (already supported), Ethereum, BNB Chain, Arbitrum, Optimism, Base, Avalanche, Polygon | Ethereum Sepolia testnet validated; Solana mainnet active |
| **High-Minting-Fee Tokens** |  Sui, Aptos                       | Solana testnet operational; Move language under evaluation |
| **Modular Chains**    | Celestia (data availability), Fuel (parallel execution) | Architectural research phase                  |

#### 2. Core Technologies for Cross-Chain Implementation
- **Unified Protocol Layer**:
  The Proof of Mint (PoM) algorithm is encapsulated as a **cross-chain smart contract module** supporting:
  - **EVM**: Solidity/Vyper versions.
  - **Move**: Sui/Aptos customized versions.
  - **SVM**: Solana native programs.

- **Cross-Chain State Synchronization**:
  Wormhole, LayerZero, or similar bridges ensure **minting progress synchronization**, maintaining consistent token supply across chains.

- **Liquidity Aggregation**:
  Unified liquidity pools across DEXs (e.g., Raydium, Uniswap, PancakeSwap) via the SLP protocol for shared depth.

#### 3. Multi-Chain Deployment Roadmap
| Phase        | Timeline       | Goals                                              |
|--------------|----------------|----------------------------------------------------|
| Q3 2024      | Solana mainnet optimization, EVM testnet (Arbitrum Sepolia) | Validate multi-chain PoM stability                 |
| Q4 2024      | Ethereum mainnet, BNB Chain, Avalanche               | Achieve full EVM ecosystem coverage                |
| Q1 2025      | Sui/Aptos mainnet deployment                        | Integrate Move ecosystem, support high-performance minting |
| Q2 2025      | Cross-chain liquidity aggregation (Wormhole-based)  | Enable minting on any mint chain with or across chains |
| Q3 2025      | Polygon, Base, Optimism integration                      | Expand to additional EVM-compatible chains          |
| Q1 2026      | Celestia + Fuel modular architecture               | Separate data availability and execution layers   |

#### 4. Advantages of Multi-Chain Strategy
- **Resilience Against Chain-Specific Risks**:
  Mitigates risks from Solana network congestion or Ethereum gas spikes, enhancing protocol resilience.
- **Ecosystem Synergy**:
  EVM chains attract DeFi users, Move chains support high-frequency minting, and Solana targets meme token communities.
  Cross-chain token standards (e.g., ERC-1155 to Mint conversion) unlock combinatorial innovation.
- **Cost Optimization**:
  Users can choose low-cost chains for minting (e.g., Base for minting, Solana for trading).

#### 5. Community Governance Role
- **Chain Deployment Voting**:
  Community members propose and vote on new chain priorities (e.g., Aptos vs. Avalanche) using governance tokens.
- **Chain-Specific Parameters**:
  Each mint chain can independently set minting parameters.

#### Summary
The **multi-chain blueprint** of flipflop.plus is a strategic move to build a **anti-fragile token economy network**. By covering EVM**, **Mint**, and **EVM** stacks, ensuring cross-chain consistency, and empowering community governance minting, PoM aims to become the first **chain-agnostic fair issuance standard**.

---

### A18. What are the platform’s governance tokens and decentralized governance mechanisms?
flipflop.plus plans to issue native governance tokens **when conditions are met** (timeline unspecified) to enable community-driven protocol upgrades and parameter governance.

- **Details**:
  - Governance tokens will allow holders to propose and vote on protocol changes, such as minting parameter adjustments or new feature integrations.
  - Specific tokenomics, distribution, and governance framework details are pending and will be announced closer to launch.

---

### A19. What is the future roadmap of flipflop.plus?
flipflop.plus’s future roadmap centers on **AI-driven minting**, **socialized entry points**, **multi-chain ecosystem**, and **closed-loop trading markets**. Below is the draft roadmap:

#### 1. AI Agent-Driven Primary Market (Minting) (Q1 2025)
- **Intelligent Parameter Minting**:
  AI Agents optimize token issuance based on on-chain data (e.g., FOMO coefficient, price staking, meme token trends), adjusting:
  - **Minting Milestone Reduction Coefficient**.
  - **Optimal Target Minting Time**.
  - Real-time Sybil attack detection.
- **AI AOperational Advisor**:
  - Provides **token issuance advice** for project creators, offering end-to-end AI services from ideation to execution with real-time guidance.
  - Recommends **minting strategies** for participants (e.g., optimal cost windows).
  - Suggests **marketing strategies** for operators and implements via AI Agents.
  - Offers **fund management advice** for value managers, executed automatically by AI Agents.
- **First Milestone**:
  - The world’s first platform to deeply integrate AI into the entire token issuance process, transitioning from **human-driven speculation** to **algorithmic equilibrium**.

#### Token2. Meme Tokengram/Discord Mini App Integration (Q3 2024)
- **Key Features**:
  - **One-Click Minting**:
    - Mint tokens directly within the Discord interface without switching wallets.
    - **Community Governance Minting**:
      - Integrated voting bots for parameter adjustment proposals via Telegram groups.
  - **Socialized Mint**:
    - Share minting achievements to channels for viral user acquisition.

#### 3. Multi-Chain Ecosystem Expansion (Minting) (2024-2025)
| Phase         | Target Minting Chains                          | Key Progress                                  |
|---------------|-------------------------|--------------------------------------|
| **Q3 2024**       | EVM testnet (Arbitrum EVM) | Validate AI Agent cross-chain adaptability     |  
| **Q4 2024**   | Ethereum Mainnet, BNB Chain, Avalanche      | Support minting token standards for semi-fungible assets |
| **Q1 2025**   | Sui, Aptos (Move Minting)| Launch parallel minting for high-concurrency Meme seasons |
| **Q2 2025**  | Celestia + Token Vault Fuel modular architecture | Reduce minting storage costs by 90% |

#### 4. Decentralized Exchange (Minting) (QX2 2025)
- **Core Features**:
  - **Liquidity Aggregation**:
    Integrates depth from DEXs like Raydium (Solana), Uniswap (Minting), and PancakeSwap (Move).
  - **Minting Support**:
    - Enables cross-chain token swaps (e.g., minting on Solana, selling on Minting).
  - **Rebase-Enhanced Tokenization**:
    - Automatically converts mint fees into Token Vault liquidity, creating anti-fragile pools.
- **Primary-Secondary Market Closed Loop**:
  - **Primary Advantage**:
    - Tokens minted on flipflop.plus minted on FlipDEX with zero listing fees.
  - **Cost-Anchored Market-Making**:
    - Adjusts market-making algorithms based on real-time minting costs to minimize price decoupling risks.

#### Summary
The roadmap of flipflop.plus emphasizes **AI-driven minting logic**, **socialized traffic minting**, **multi-chain scenario coverage**, and **DEX-closed tokenization** to become the “operating ecosystem for token economics” in Web3. Through vertical tech stack integration and horizontal ecosystem expansion, it aims to break the **impossible triad** of **fairness**, **efficiency**, and **scale**.

---

## B. Launch (Continued)

### B1. (Continued) How to mint a token?
**Minting in 6 Steps, Completed in ~1 Minute**:
1. **Minting Testnet**:
   - Visit [https://test.flipflop.plus](https://test.flip), connect a Solana wallet (e.g., Phantom, Solflare).
2. **Create Token Minting**:
   - Click **Minting Token** in the left menu.
   - Enter token name, minting, symbol, and icon (URL or upload).
   - *Optional*: Set minting start time or social media links.
3. **Select Model**:
   - **Standard Minting**: 100M token supply, suited for tokens.
   - **Meme Token**: 1B token supply, ideal for meme coins.
4. **Tokenize Mint**:
   - Click **Create Token**, sign the wallet mint transaction (pay testMint gas fee).
5. **Await Confirmation**:
   - Token contract deployment completes in ~30 seconds.
6. **View Mint Token**:
   - Go to **Tokens → My Deployment**, check token address, minting progress, and liquidity settings.

**Note**:
- TestMint tokens have no real value and are for minting validation only.

---

### B2. What are the differences between Standard and Minting modes, and how to choose?
Mintingflipflop.plus offers two pre-configured tokenomics models optimized for different scenarios. Below are the key differences and selection strategies:

#### **1. Differences Comparison**

##### **Differences**  
| **Parameter**                    | **Standard Modeing**              | **Minting Mode**                  |
|----------------------------|------------------------------------|-----------------------------------|
| **Max Supply**            | 100,000 tokens,000 (1M)            | Tokens (1B)000,000)           |
| **Minting Coefficient**       | 50% (50% supply reduction per milestone) | | 75% (25% reduction per milestone token) |
| **Total Minting Cost at Graduation** | 1,000–4,457 SOL tokens               | 250–1,000 SOL tokens               |
| **Tokens Minted at Mint**    | Minted50,000 tokens,000 (50%)          | 250 (25% tokens of) total supply) |
| **Token Price Range**      | **0.00008 SOL** | **0.000008 SOL**               |
| **Single Mint Cost**         | 0.2 SOL tokens                        | 0.01 SOL (95% lower token)           |
| **Target Token Per Checkpoint** | 200,000 tokens                    | **1,000,000 tokens** (5x Standard token) |

##### **Similarities**  
| **Parameter**                | **Tokenization**                  |
|----------------------------|------------------------------------|
| **Target Mint Time**        | ~5.79 days tokens                        |
| **Initial Minting Amount**    | 10,000 tokens                      |
| **Target Mintingstone**       | 1 (graduates upon completion)      |
| **Checkpoints per Milestone**| 250 tokens                         |
| **Target Checkpoint Time**  | 2000 seconds (~33 minutes)         |
| **Liquidity Injection Ratio**| 20% of minting fees to DEX         |

#### **2. Tokenomics Differences**
- **Standard Mode Minting**:
  - **Strong Deflationary Model**: Halves token supply per milestone, ideal for **scarcity-driven value appreciation** (e.g., DeFi protocols, utility tokens).
  - **High-Cost Speculation Filter**: 0.2 SOL per mint deters short-term speculation, attracting long-term holders.
  - **Use Cases**: DAO governance tokens, infrastructure credits.

- **Minting Mode**:
  - **High-Circulation Design**: 1B supply with low unit price (from 0.000004 SOL), suited for **community-driven viral spread**.
  - **Ultra-Low Entry Barrier**: 0.01 SOL per mint (1 SOL mints 100K tokens), appealing to mass participation.
  - **Use Cases**: Animal-themed coins, social meme tokens.

#### **3. Selection Strategy**
| **Dimension**     | **Standard Mode Minting**            | **Minting Mode**                  |
|------------------|-------------------------------|-----------------------------------|
| **Project Goal** | Long-term ecosystem building, value storage | Short-term hype, community culture spread |
| **Token Distribution**| Controlled circulation, anti-inflation | Broad distribution, rapid holder base growth |
| **Community Type**| Developers, institutional investors | Retail, KOLs, meme enthusiasts |
| **Risk Appetite**| Low volatility, downside resistance | High volatility, high-growth potential |

#### **4. Operational Recommendations**
- **Project Creators**:
  - **Fundraising + Ecosystem Control** → Choose Standard mode for deflationary value growth.
  - **Community Bootstrapping** → Select Meme mode for low-cost viral adoption.
- **Users**:
  - **Stable Returns** → Mint in Standard mode (0.2 SOL/mint), focus on long-term value.
  - **Speculative Gains** → Mint in Meme mode (0.01 SOL/mint), leverage high volume for upside.

---

### B3. If neither Standard nor Meme mode meets my needs, can I customize token parameters?
Yes, customization is possible. Please contact the platform for assistance.

---

### B4. How are the expected token price and total minting cost calculated during token launch, and why is it a range?
The token price and total minting cost range on flipflop.plus are determined by the **liquidity pool initialization model** and **FOMO coefficient dynamics**. Below is the detailed calculation process:

#### 1. Core Formulas and Parameters
##### Liquidity Pool Initialization Price Formula
Per the platform’s design, the initial token price is calculated as:

$$
\text{Price} = \frac{0.90 \cdot \text{TotalFee}}{\text{InitLiquidity}}
$$

- **TotalFee**: Total minting fees (after deducting 5% protocol fee and 0–5% referral fee, 90–95% is injected into the liquidity pool; 90% is used as the lower bound here).
- **InitLiquidity**: Initial token amount in the liquidity pool.

##### Key Parameter Definitions
| Parameter | Description                              | Standard Mode Value | Meme Mode Value     |
|-----------|------------------------------------------|---------------------|---------------------|
| $P_0$     | Single minting fee                       | 0.2 SOL             | 0.01 SOL            |
| $T_0$     | Target minting amount per Checkpoint     | 200,000 tokens      | 1,000,000 tokens    |
| $M_0$     | Base minting amount per operation        | 10,000 tokens       | 10,000 tokens       |
| $C$       | Number of Checkpoints per Milestone      | 250                 | 250                 |
| $E$       | Total Milestones (default 1 at launch)   | 1                   | 1                   |
| $f$       | Milestone decay coefficient              | 50%                 | 75%                 |
| $r_l$     | Liquidity pool ratio (of total supply)   | 20%                 | 20%                 |

#### 2. Step-by-Step Calculation Example (Standard Mode)
##### Step 1: Calculate Initial Liquidity Pool Token Amount (InitLiquidity)

$$
\text{InitLiquidity} = C \cdot T_0 \cdot r_l \cdot \frac{1 - f^E}{(1 - f)(1 - r_l)}
$$

Parameters: $C=250$, $T_0=200,000$, $r_l=0.2$, $f=0.5$, $E=1$

$$
\text{InitLiquidity} = 250 \cdot 200,000 \cdot 0.2 \cdot \frac{1 - 0.5^1}{(1 - 0.5)(1 - 0.2)} = 12,500,000 \ \text{tokens}
$$

##### Step 2: Calculate Total Minting Fee Range (TotalFee)

$$
\text{TotalFee} \in \left[ \frac{P_0 \cdot T_0}{M_0} \cdot C_e, \ \frac{P_0 \cdot T_0}{M_0} \cdot 101 \cdot (1.01^{C_e} - 1) \right]
$$

Where $C_e = E \cdot C = 250$
- **Lower Bound (No FOMO Increase)**:

$$
\text{TotalFee}_{\text{min}} = \frac{0.2 \cdot 200,000}{10,000} \cdot 250 = 4 \cdot 250 = 1,000 \ \text{SOL}
$$

- **Upper Bound (FOMO Increases 1% per Checkpoint)**:

$$
\text{TotalFee}_{\text{max}} = \frac{0.2 \cdot 200,000}{10,000} \cdot 101 \cdot (1.01^{250} - 1) \approx 4,457 \ \text{SOL}
$$

##### Step 3: Calculate Initial Price Range
- **Lowest Price ($P_{\text{low}}$)**:

$$
P_{\text{low}} = \frac{0.90 \cdot \text{TotalFee}_{\text{min}}}{\text{InitLiquidity}} = \frac{0.90 \cdot 1,000}{12,500,000} \approx 0.000072 \ \text{SOL/token}
$$

- **Highest Price ($P_{\text{high}}$)**:

$$
P_{\text{high}} = \frac{0.90 \cdot \text{TotalFee}_{\text{max}}}{\text{InitLiquidity}} = \frac{0.90 \cdot 4,457}{12,500,000} \approx 0.000321 \ \text{SOL/token}
$$

#### 3. Reasons for the Range
1. **FOMO Coefficient Dynamics**:
   - If minting is **faster than target**, the FOMO coefficient increases by up to 1% per Checkpoint, leading to exponential fee growth.
   - If minting is **on target**, the FOMO coefficient remains 1, yielding the minimum fee.
2. **Liquidity Pool Ratio Constraint**:
   The 20% initial liquidity ratio anchors the token supply, with price scaling linearly with total fees.

#### 4. Design Significance and User Strategies
- **Price Anchoring**: Algorithmic constraints prevent issuer manipulation.
- **Participation Strategies**:
  - **Low-Risk Users**: Mint at target pace for lowest cost (e.g., 0.000288 SOL).
  - **High-Risk Users**: Mint early, paying higher costs (up to 0.001283 SOL) for early tokens.

#### Summary
flipflop.plus uses **rigid liquidity ratios** and **dynamic fee models** to define a quantifiable price range, ensuring fairness in decentralized issuance while preserving flexibility for market dynamics.

---

### B5. How to set the name and symbol during token launch, and what are the restrictions?
When issuing a token on flipflop.plus (Solana-based), the name and symbol must adhere to the following strict rules:

#### 1. Token Name Restrictions
- **Length Limit**: Up to **32 characters** (including spaces and symbols).
- **Allowed Characters**:
  - ASCII letters (A-Z, a-z)
  - Numbers (0-9)
  - ASCII punctuation (e.g., `!`, `@`, `#`, `$`, `%`)
  - Single emoji (e.g., 🚀, 🌟)
  - Spaces (no consecutive spaces)
- **Prohibited**:
  - All spaces (e.g., "  ")
  - Non-ASCII characters (e.g., Chinese, Japanese)
  - Consecutive spaces (e.g., "  ")

**Valid Examples**:
- `Dragon Coin 🐉`
- `DeFi_Protocol_2023`
- `Meme Token!`

**Invalid Examples**:
- `  ` (all spaces)
- `比特币` (non-ASCII)
- `Hello  World` (consecutive spaces)

#### 2. Token Symbol Restrictions
- **Length Limit**: Up to **10 characters** (recommended 3–5).
- **Allowed Characters**:
  - ASCII letters (A-Z, a-z)
  - Numbers (0-9)
  - Single emoji (e.g., 🚀, 💎)
- **Prohibited**:
  - Spaces, punctuation, or other special characters (e.g., `$`, `@`)
  - Multiple emojis (e.g., 🚀🌟)

**Valid Examples**:
- `DRGN`
- `M202`
- `🚀`

**Invalid Examples**:
- `DEFI$` (punctuation)
- `BT C` (space)
- `🚀🌟` (multiple emojis)

#### 3. Metadata URI Rules
- **Format**: Must start with `https://`.
- **Length Limit**: Up to **200 characters**.

**Valid Example**:
- `https://node1.irys.xyz/F7qkMeZGbDwZbbo6E6XkOahuUlGEcXElWgGnH5zm4z`

**Invalid Examples**:
- `ftp://example.com/icon.png` (unsupported protocol)
- `data:image/png;base64,...` (non-https/IPFS)

#### 4. Metadata Format
The metadata URI resolves to a JSON object like:
```
{
  name: "Satoshi",
  image: "https://gateway.irys.xyz/fzuVcFKiJaeuRzz3JSWXs3aC3JPxYyggDQCzel_uv",
  symbol: "SATO",
  description: "Hello Satoshi",
  creator: {
    name: "Flipflop",
    site: "https://app.flipflop.plus"
  },
  extensions: {
    website: "",
    twitter: "https://x.com/shanghai",
    telegram: "",
    discord: ""
  }
}
```
- **Required Fields**: `name`, `image`, `symbol`, `description`.
- **Optional Fields**: `creator`, `extensions` (customizable).

#### 5. Can Name and Symbol Be Duplicated?
- **Name + Symbol Uniqueness**: The combination must be unique; different names with the same symbol are allowed.
- **Symbol Uniqueness**: Symbols cannot be duplicated and are case-sensitive.
- **Name Case Sensitivity**: Names are case-insensitive; non-identical case variations can be reused.

**Valid Examples**:
- `SATO` and `Sato` are considered duplicates; first issuer claims it.
- `Satoshi token` and `satoshi token` are not duplicates.

**Design Intent and Guidance**:
- **Prevent Confusion**: Unique symbols avoid imitation scams.
- **Balance Flexibility**: Allows reuse of short symbols with distinct names.
- **Recommendations**:
  - Check name/symbol availability via Solana explorers (e.g., Solscan) before issuance.
  - If a symbol is taken, use suffixes (e.g., DRGN2) or more specific names (e.g., Dragon Protocol).

---

### B6. Where are token metadata and image data stored, and can they be lost?
Token metadata and image data for tokens issued on flipflop.plus are stored with decentralization and permanence in mind.

#### Storage Locations
1. **Primary Storage: Irys Network**
   - Metadata and images are preferentially stored on the **Irys Network**, a decentralized storage solution optimized for efficiency and permanence.
   - **Features**: Irys integrates with Arweave for permanent storage while optimizing upload speed and cost. It supports fast transaction confirmation, ideal for frequent data uploads like token metadata.

2. **Fallback Storage: Arweave Network**
   - If Irys is unavailable, data is stored on the **Arweave Network**, a decentralized permanent storage protocol.
   - **Features**: Arweave uses a “pay once, store forever” model, with data stored across multiple nodes for long-term availability.

#### Overview
- **Irys Network**:
  - Irys (formerly Bundlr) is a Layer 2 solution on Arweave, enhancing decentralized storage efficiency. It batches uploads and optimizes fees while ensuring permanence.
  - **Advantages**: Fast uploads, low costs, seamless Arweave integration.
- **Arweave Network**:
  - Arweave uses “Blockweave” technology for permanent data storage, with a one-time fee ensuring data availability for centuries.
  - **Advantages**: Tamper-proof, no recurring fees, long-term reliability.

#### Can Data Be Lost?
- **No Loss**: Both Irys and Arweave use decentralized permanent storage. Once uploaded, data is distributed across multiple nodes, eliminating reliance on centralized servers (e.g., AWS) and mitigating loss risks.
- **Irys Assurance**: Irys leverages Arweave’s permanence, ensuring data accessibility even if Irys faces issues.
- **Arweave Assurance**: Designed for 200+ years of storage, with economic incentives for nodes to maintain data.

---

### B7. Can I issue multiple tokens?
Yes, you can issue multiple tokens as long as they comply with the name and symbol naming rules.

---

### B8. How do I find all the tokens I’ve issued?
In the flipflop.plus app, navigate to **Tools → My Deployments** to view all tokens you’ve issued.

---

### B9. How secure are tokens issued via flipflop.plus, and how do token validators score them?
flipflop.plus enforces strict security via smart contracts for Solana SPL tokens. Below is the permission management mechanism:

#### 1. Mint Permission (Token Issuance)
- **Initial Setup**:
  - **Mint Permission Discarded**: Permission is revoked immediately upon token creation, ensuring fixed supply.
  - **Pre-Minted Tokens**: All tokens are minted at creation and stored in the **Mint Token Vault (PDA account)**, controlled by the flipflop.plus program.
  - This is called the **Pre-Mint + PDA Custody** model.
- **Security Rules**:
  - **No Additional Issuance**: No one, including the issuer, can mint more tokens.
  - **Contract-Driven Airdrops**: Tokens are distributed solely via program logic, preventing manual interference.
  - Supports DEXs for tokens with restricted mint permissions not yet discarded.

#### 2. Freeze Permission (Token Freezing)
- **Initial Setup**:
  - **Freeze Permission Discarded**: No account can freeze tokens.
- **Security Impact**:
  - User assets are fully secure, immune to freezing by issuers or third parties.
  - Supports DEXs for tokens with restricted freeze permissions not yet discarded.

#### 3. Close Permission (Token Closure)
- **Initial Setup**:
  - Issuers retain **Close Permission** with restrictions:
    - **Before Minting or Full Refund**: Tokens can be closed, reclaiming account rent.
    - **After Minting Starts**: Close permission is permanently disabled.
- **Design Intent**:
  - Prevents issuers from destroying tokens post-fundraise.

#### 4. Metadata Update Permission
- **Controlled Flexibility**:
  - **Editable**: Issuers can update metadata fields except `name` and `symbol` (e.g., social links, banners).
  - **Lock Option**: A **one-click disable** feature permanently locks metadata.
- **Security Boundary**:
  - Prevents `name`/`symbol` edits to avoid impersonation attacks.

#### 5. Permission Comparison (flipflop.plus vs. Traditional SPL Tokens)
| Permission       | Traditional SPL Tokens         | **flipflop.plus**              |
|------------------|-------------------------------|-------------------------------|
| **Mint**         | Issuer may retain minting rights | Permanently discarded, fixed supply |
| **Freeze**       | Issuer can freeze assets      | Permanently discarded, no freezing |
| **Close**        | Issuer can close anytime      | Only pre-minting; disabled post-minting |
| **Metadata**     | Issuer can edit all fields    | Non-critical fields editable, lockable |

#### 6. Security Design Summary
1. **Anti-Inflation**: Discarded mint permission eliminates over-issuance risks.
2. **Anti-Freezing**: Assets are solely controlled by private keys.
3. **Anti-Rug Pull**: Close permission disabled post-minting ensures token persistence.
4. **Transparent Verification**: All permission changes are on-chain, verifiable via explorers (e.g., Solscan).

---

### B10. How are tokens minted if the mint permission is discarded?
flipflop.plus uses a **Pre-Mint + PDA Custody** model to mint tokens securely despite discarded mint permissions. Below is the mechanism:

#### 1. Two-Stage Token Issuance
##### Stage 1: Pre-Minting and Permission Destruction
- **Pre-Mint All Tokens**:
  The entire token supply (e.g., 100M) is minted at creation and transferred to the **Mint Token Vault (PDA account)**.
- **Discard Mint Permission**:
  The smart contract permanently revokes mint permission post-pre-mint, ensuring no further issuance.

##### Stage 2: Dynamic Distribution (Airdrop)
- **PDA Custody**:
  The **Mint Token Vault** is controlled by the flipflop.plus smart contract, distributing tokens as follows:
  - During minting, the contract airdrops the allocated amount to user accounts.
  - Tokens for liquidity pools are airdropped to the **Token Vault** per the liquidity ratio.
  - **No Manual Access**: No entity, including the issuer, holds the PDA’s owner key, preventing direct token transfers.

#### 2. Key Security Features
| Feature              | Description                                                           |
|----------------------|-----------------------------------------------------------------------|
| **Hard Supply Cap**  | Pre-minting locks total supply at the initial setting (e.g., 100M).    |
| **Decentralized Distribution** | Smart contract logic solely handles token allocation, avoiding human manipulation. |
| **Immutable PDA**    | The Mint Token Vault lacks a private key, ensuring no unauthorized withdrawals. |

#### 3. Mint Token Vault Destruction (Post-Graduation)
- **Purpose**:
  Allows the community to burn undistributed tokens in the Vault after graduation (Target Milestone), enabling:
  - **Supply Reduction**: Increases token scarcity.
  - **Minting Halt**: Stops further distribution.
  - **User Tokens Unaffected**: Only unallocated tokens are burned.
- **Example**:
  A meme coin with a 1B supply has 250M distributed at graduation. The community burns the remaining 750M in the Vault, reducing the total supply to 250M and halting minting.

#### 4. Comparison with Traditional Models
| Feature              | Traditional SPL Tokens                | **flipflop.plus**                     |
|----------------------|---------------------------------------|---------------------------------------|
| **Issuance Risk**    | Issuer may retain minting rights       | Pre-mint + permission discard locks supply |
| **Distribution Control** | Relies on issuer manual transfers     | Smart contract automated airdrops      |
| **Transparency**     | Depends on issuer trust               | All distribution on-chain, verifiable  |

#### 5. User Verification
- **Blockchain Explorer**:
  Use Solscan to verify:
  1. **Mint Permission Status**: Shows `null` (discarded).
  2. **Mint Token Vault Balance**: Tracks undistributed tokens.
  3. **Burn Records**: Transparent post-graduation burns.

#### Summary
flipflop.plus ensures secure minting via **pre-minting**, **permission destruction**, and **PDA custody**, eliminating issuance risks while enabling fair distribution. The optional post-graduation burn empowers community-driven deflation, embodying decentralized principles.

---

### B11. Why does the top account in the token holder ranking on blockchain explorers hold so many tokens?
This account is the **Mint Token Vault** address. Due to the **Pre-Mint + PDA Custody** model, it holds the entire initial token supply at creation, not the actual distributed amount.

---

### B12. Can I delete a previously issued token on-chain if I need to make changes after issuance?
#### 1. Editable Content
- **Allowed Changes**:
  - **Metadata**: Social links (Twitter, Discord), banner images, descriptions, website URLs.
- **Prohibited Changes**:
  - **Name**, **Symbol**, **Mint Account** (core token identity).

#### 2. How to “Delete” an Issued Token
If no minting has occurred (i.e., no user participation), you can:
1. **Close and Reclaim Resources**:
   - Use the `Close` instruction to shut down associated accounts (e.g., Metadata) except the Mint account.
   - Path: **Tools → My Deployments → Select Token → Close Token**.
   - **Reclaim SOL Rent**: Closing accounts refunds some storage rent.
2. **Redeploy New Token**:
   - Since the Mint account cannot be deleted, create a new token with a different Name/Symbol.
   - The old token persists on-chain but is non-functional (no minting due to closed accounts).

#### 3. Key Restrictions and Notes
| Scenario                  | Feasible?                | Solution                              |
|---------------------------|--------------------------|---------------------------------------|
| **Edit Name/Symbol**      | ❌ No                    | Create new token                      |
| **Delete Mint Account**   | ❌ No (Solana limitation) | Old Mint persists but can be abandoned |
| **Token Already Minted**  | ❌ Cannot delete         | Adjust rules via community governance (e.g., burn) |

#### 4. Operational Recommendations
- **Pre-Issuance Validation**: Ensure Name/Symbol are correct to avoid errors.
- **Metadata Flexibility**: Update social links or banners for project evolution.
- **Emergency Handling**: Close old token immediately if no minting has occurred and redeploy.

#### 5. Design Intent
- **Security**: Prevents malicious identity changes (e.g., impersonation scams).
- **Flexibility**: Metadata updates support project growth.
- **Transparency**: All changes are on-chain, preventing centralized manipulation.

**Summary**: flipflop.plus balances security and flexibility, allowing metadata updates while locking core attributes. Tokens cannot be fully deleted, but pre-minting closure and redeployment provide practical solutions.

---

### B13. How to estimate the planned minting completion time and the number of operations per hour or minute?
The minting time estimation on flipflop.plus is based on a **geometric series model** and **dynamic decay mechanism**. Below is the mathematical derivation:

#### 1. Total Time Formula
Each Milestone’s time is $C \cdot t_{\text{check}}$ ($t_{\text{check}}$ is the target Checkpoint time), so total time is:

$$
T_{\text{total}} = E \cdot C \cdot t_{\text{check}}
$$

#### 2. Case Study 1 (Standard Mode)
- **Parameters**: $C = 250$, $t_{\text{check}} = 2000$ seconds, $E = 1$
- $T_{\text{total}} = 250 \cdot 2000 \cdot 1 = 500,000$ seconds ≈ **5.79 days**

#### 3. Case Study 2 (Meme Mode)
- **Parameters**: $C = 250$, $t_{\text{check}} = 2000$ seconds, $E = 1$
- $T_{\text{total}} = 250 \cdot 2000 \cdot 1 = 500,000$ seconds ≈ **5.79 days**

---

### B14. Can social information in token metadata be updated, and is there a fee for updating metadata?
Token metadata on flipflop.plus is editable, allowing updates to social information (e.g., Twitter, Discord).

#### 1. Update Process
- **User Action**:
  Navigate to **Tools → My Deployments → Select Token → Update Metadata**.
- **Smart Contract**:
  The contract verifies the user’s **Update Metadata** permission; without it, updates are blocked.

#### 2. Fees
- **Update Fee**: 0.1 SOL.
- **Tip**: Set metadata correctly during issuance to minimize update fees. Note that the token homepage banner can only be updated post-issuance.

---

### B15. How to prevent arbitrary metadata updates from negatively impacting the community?
Issuers can permanently disable metadata updates to prevent misuse.

#### 1. Disable Update Permission
- **User Action**:
  Go to **Tools → My Deployments → Select Token → Authority → Revoke Metadata Update**.
  - **Mutable**: Metadata can be updated.
  - **Immutable**: Metadata is locked.

---

### B16. Can I set a token’s minting start time?
Yes, during token issuance, you can specify a future start time for minting. Before this time, users can generate URC referral codes as usual.

---

## C. Mint and Refund

### C1. How to find token information?
You can access token information and the minting page via four methods:
1. **QR Code Scan**:
   Scan the dynamic QR code in the bottom-right of referral images to jump to the minting page.
2. **Social Media Links**:
   Click referral links on social platforms (Twitter, Telegram) for direct access.
3. **Platform Search**:
   Enter the token name, contract address, or symbol (case-sensitive) in the flipflop.plus app’s homepage search bar.
4. **Social Network Tracking**:
   Follow project or KOL profiles in the app’s “Community” section for real-time token updates.

---

### C2. How to mint tokens?
**Minting Process (3 Steps)**:
1. **Access Token Page**:
   Locate the target token page via any method above.
2. **Enter Verification**:
   Click “Mint,” then input a valid URC (User Referral Code) in the pop-up.
3. **Confirm Minting**:
   Connect your wallet, click “Start Minting,” pay the gas fee, and sign the transaction. Tokens are allocated to your wallet in real-time.

> **Tip**: Ensure sufficient SOL balance and a stable network before minting.

---

### C3. Is a URC required to mint tokens, and how do I obtain one?
#### URC Necessity
The **User Referral Code (URC)** is mandatory for minting, serving dual purposes:
1. **Anti-Automation**:
   On-chain encryption blocks bot bulk acquisition.
2. **Community Trust**:
   Builds a robust human community via real-user referrals.

#### URC Acquisition
If you lack a URC, obtain one via:
- **Community Engagement**:
  Request a URC from admins or active members in project communities (Telegram, Discord).
- **Offline Events**:
  Attend project AMAs or meetups to receive high-quality URCs.

#### Off-Chain Anti-Eavesdropping
URCs use end-to-end encryption, distributed via off-chain channels (e.g., images, DMs), preventing on-chain data interception and ensuring human usage.

[NEXT PART](FAQ_3.md)