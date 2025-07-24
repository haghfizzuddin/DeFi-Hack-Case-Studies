# Cetus AMM Overflow Exploit (May 2025)

## Exploit overview

On May 14, 2025, **Cetus Protocol**, a concentrated liquidity AMM on the Sui blockchain, suffered a critical arithmetic overflow bug that allowed attackers to mint excessive liquidity tokens and drain the pool. The attack resulted in a theft of approximately **$1.9 million** worth of assets, primarily SUI and USDC. The exploit occurred within a vulnerable smart contract handling tick-based liquidity ranges.

The stolen funds were sent to the attacker’s wallet: **`0x53093F4D77c3fAA0aEfCc13e7746f3E1eA517c56`** (cross-bridge withdrawal address).

## Root cause

The vulnerability was due to **unchecked multiplication** when calculating liquidity deltas across price ticks. Specifically:

- **Unchecked arithmetic**: A multiplication operation between tick spacing and loop counters led to an **integer overflow**, allowing an attacker to bypass range checks and expand their liquidity position far beyond the intended limits.
- **Lack of overflow protection**: The vulnerable contract failed to use safe math operations or Sui-native arithmetic guards.
- **No cap on tick traversal**: The logic assumed a valid price range traversal but did not impose a strict upper bound, enabling the loop to misbehave when overflowed.

## Attack path

1. **Craft overflowed liquidity range**: The attacker passed crafted parameters to a liquidity position minting function, causing the tick loop to overflow due to unchecked arithmetic.
2. **Bypass price boundaries**: The overflowed tick range allowed the attacker to simulate massive virtual liquidity without depositing proportional assets.
3. **Drain the pool**: The attacker withdrew tokens based on the inflated liquidity position, extracting millions in SUI and USDC.
4. **Bridge and obfuscate**: The attacker transferred the funds to Ethereum and then through multiple bridge routes to launder the proceeds.

## On-chain indicators

- **Abnormal tick range parameters**: The exploit transaction involved unusually high tick values far exceeding realistic pool bounds.
- **Excessive liquidity minted**: A liquidity position was created with vastly disproportionate size compared to the deposited token amount.
- **Rapid token withdrawal**: A single address executed a high-frequency withdrawal immediately after the minting call.
- **Bridging to Ethereum**: The attacker quickly routed funds to Ethereum via cross-chain bridges, using `0x53093F4D77c3fAA0aEfCc13e7746f3E1eA517c56`.

## Lessons learned

- **Always use safe math or overflow-checked arithmetic**: Even in newer smart contract languages like Move (Sui), developers must explicitly ensure arithmetic safety when performing loop-bound math.
- **Enforce tick bounds**: Liquidity provisioning logic should strictly validate tick ranges against pool limits and reject unrealistic inputs.
- **Test edge-case tick paths**: Fuzz testing should include upper-bound ranges and extreme arithmetic scenarios to catch tick boundary violations.
- **Monitor large position mints**: Real-time monitoring of liquidity provisioning patterns could detect unusual ranges or mint attempts.

