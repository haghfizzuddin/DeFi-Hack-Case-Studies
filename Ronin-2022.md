# Ronin Bridge Validator Compromise (March 2022)

## Exploit overview

On March 23, 2022, the Ronin Network—a side-chain used by Axie Infinity—experienced a catastrophic bridge hack. Attackers drained 173,600 ETH and 25.5 million USDC, totaling approximately $600 million, from the Ronin–Ethereum bridge. The exploit remained undetected for six days, only coming to light when a user was unable to withdraw 5,000 ETH on March 29 2022.

## Root cause

Ronin uses a multi-signature bridge controlled by nine validator nodes. A withdrawal requires five of these nodes to sign a transaction. The root cause of the hack was a compromise of validator private keys combined with lax access control:

* **Centralisation:** Four of the nine validators were operated by Sky Mavis (Axie's developer). This meant Sky Mavis effectively controlled half of the required signatures.
* **Delegated privileges:** In 2021 the Axie DAO temporarily allowed Sky Mavis to sign transactions on its behalf to cope with high traffic. Although the program ended, the allowlist was never revoked.
* **Insufficient monitoring:** There was no on-chain or off-chain system to detect large, unusual withdrawals. The transfer of hundreds of millions of dollars went unnoticed until a user complained.

## Attack path

1. **Compromise validators.** The attacker gained access to private keys for four Sky Mavis validators and a fifth validator operated by Axie DAO, exploiting the stale delegation arrangement.
2. **Construct fraudulent withdrawals.** With control over five validators, the attacker created and signed two withdrawal transactions from the bridge contract, authorising the transfer of 173,600 ETH and 25.5 million USDC to attacker-controlled addresses.
3. **Exfiltrate funds.** The assets were removed from the bridge and moved into wallets controlled by the attacker. Because the bridge was under-monitored, the transfers were not detected for several days.

## On-chain indicators

* **Validator signature anomalies:** Two large withdrawals were authorised by the same set of validators, including a normally inactive Axie DAO node. Monitoring for unusual validator participation could have flagged the activity.
* **Large, uncharacteristic transfers:** Transfers of hundreds of millions of dollars worth of ETH and USDC out of the bridge contract occurred within a short period.
* **Absence of corresponding deposits:** There were no large deposits into the bridge to offset the withdrawals, which should have triggered an alert.

## Lessons learned

The Ronin hack underscores several security principles for cross-chain bridges:

* **Decentralise validator sets.** Relying on a small number of validators under one organisation undermines the purpose of a multi-signature scheme. Validators should be distributed among independent parties.
* **Revoke temporary privileges.** Temporary arrangements, such as delegating a validator's signing authority, must be revoked as soon as they expire.
* **Enforce least privilege.** Validators should only have the minimal rights required to perform their duties. Any special permissions should be time-bound and well monitored.
* **Implement real-time monitoring.** Bridges should track large withdrawals and unusual validator patterns. Automated alerts could have caught the theft within minutes rather than days.
* **Rotate and secure keys.** Regular key rotation and hardened operational security help prevent attackers from compromising validator keys in the first place.
