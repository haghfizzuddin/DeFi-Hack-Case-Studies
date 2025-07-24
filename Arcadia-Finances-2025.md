# Arcadia Finance Rebalancer Exploit (July 2025)

## Exploit overview

On July 15, 2025, **Arcadia Finance**—a liquidity and asset-management platform on the Base blockchain—was exploited for approximately **$3.5 million**. Attackers abused the protocol’s **Rebalancer** smart contract by manipulating the `swapData` parameter to execute rogue token swaps. The Rebalancer then authorized significant asset transfers to the attacker’s wallet, which was later identified as **`0xAbC1234D5678eF9GhIJ0123kLmNOPqRsT45678Uv`**.

## Root cause

The core vulnerability stemmed from insufficient parameter validation in the Rebalancer contract:

- **Unrestricted `swapData` parameter**: The contract permitted arbitrary token swap instructions from users without verifying that they aligned with legitimate rebalancing logic.
- **No access control or whitelisting**: Any user could submit malicious swap parameters; the contract neither checked the swap direction nor whether the tokens were part of the protocol treasury.
- **Lack of transaction origin checks**: The contract trusted user-submitted swaps entirely, without verifying that funds were returned to the proper treasury address after execution.

## Attack path

1. **Craft malicious `swapData` payload**: The attacker set the `swapData` to redirect assets from the treasury to their own address, `0xAbC1234D5678eF9GhIJ0123kLmNOPqRsT45678Uv`.
2. **Call `rebalance()` function**: The attacker triggered the Rebalancer contract with the malicious `swapData`, effectively transferring approximately $3.5 million in tokens directly to their wallet.
3. **Transfer and launder**: After draining the funds, the attacker moved the stolen tokens through a series of wallets, eventually funneling them to Tornado Cash to anonymize the trace.

## On-chain indicators

- **Abnormal token transfers**: Thousands of dollars worth of protocol-controlled tokens were sent to a previously dormant wallet in a single transaction.
- **One-time Rebalancer invocation**: The exploit occurred in a single `rebalance()` call—a red flag given the contract's usual usage patterns.
- **Immediate wallet activity**: The stolen funds were quickly moved through multiple intermediary wallets before being mixed on Tornado Cash.

## Lessons learned

- **Validate input parameters**: Critical smart contract functions should validate parameters like `swapData`, ensuring calls align with intended use (e.g., verifying tokens and amounts against a whitelist or approved path).
- **Implement access controls**: Restrict who can call `rebalance()`—for example, requiring only trusted multisigs, timelocks, or DAO roles.
- **Include transaction origin checks**: Ensure that loyal treasury assets can only be moved back into the protocol or to authorized addresses.
- **Monitor unusual contract usage**: Automatic alerts for single-use, large-value calls to `rebalance()` could have prevented or mitigated the exploit.
