# Beanstalk Governance Attack (April 2022)

## Exploit overview

In April 2022 the Beanstalk stablecoin protocol was drained of roughly $181 million in collateral. Attackers abused the protocol's decentralized governance system to push through two malicious proposals and siphoned funds from the treasury. The flash-loan-assisted governance attack allowed the perpetrator to control a supermajority of votes, execute an emergency function and transfer the assets to their own address【490825864822557†L71-L99】. Although the treasury lost $181 million, the attacker ultimately walked away with around $76 million after repaying the flash loans【490825864822557†L71-L99】.

## Root cause

The core vulnerability lay in Beanstalk's governance design. Voting power was proportional to the amount of tokens deposited into the protocol's *Diamond* contract. The project included an `emergencyCommit` function that allowed proposals to be enacted with a 2/3 super-majority after a 24-hour waiting period【490825864822557†L77-L95】. Because there were no limits on the size of deposits or safeguards against flash-loaned deposits, an attacker could temporarily borrow large sums, deposit them, gain voting control and push through a malicious proposal. This logic had not been audited and lacked monitoring【490825864822557†L102-L113】.

## Attack path

1. **Design and deploy malicious proposals.** The attacker created two proposals (#18 and #19) that looked like ordinary governance actions but contained code to transfer Beanstalk's reserves to the attacker's address and a donation address【490825864822557†L82-L85】. They waited for the 24-hour timelock to expire.
2. **Acquire voting power.** After the waiting period, the attacker used a series of flash loans (reportedly totalling around $1 billion in crypto assets) to deposit enough governance tokens into the Diamond contract to control approximately 79 % of the voting power【490825864822557†L87-L95】【612784540889037†L135-L139】. This far exceeded the 2/3 threshold required to trigger `emergencyCommit`.
3. **Execute the proposals.** With a supermajority secured, the attacker called `emergencyCommit` to execute the proposals【490825864822557†L77-L95】. The proposals drained the reserves and sent the stolen tokens to both the attacker and a donation address【490825864822557†L82-L99】.
4. **Repay loans and launder funds.** After receiving the funds, the attacker repaid the flash-loan providers and then routed the proceeds through Tornado Cash to obfuscate the trail【612784540889037†L127-L139】.

## On-chain indicators

Several on-chain signals preceded and accompanied the attack:

* **Large governance deposits:** Immediately prior to the `emergencyCommit` call, a single transaction deposited an unusually large amount of governance tokens into the Diamond contract, pushing the attacker's voting share above the super-majority threshold【612784540889037†L110-L139】.
* **Multiple malicious proposals:** Two proposals were introduced in quick succession with similar payloads. Monitoring unusual proposal contents could have raised alarms.
* **Sudden value transfer:** A single execution transferred hundreds of millions of dollars worth of tokens from Beanstalk to addresses not previously associated with the protocol.
* **Flash-loan repayment and mixing:** Within the same block, the attacker repaid the flash loan and routed remaining funds to Tornado Cash【612784540889037†L127-L139】.

## Lessons learned

The Beanstalk incident highlights several important security lessons for DeFi projects:

* **Limit instantaneous voting power.** Governance systems should restrict how much influence can be obtained in a single transaction or block. Time-weighted voting or blocking flash-loaned deposits can mitigate similar attacks.
* **Audit and monitor governance functions.** Critical functions such as `emergencyCommit` should be thoroughly audited and monitored. Long timelocks and off-chain alerts can provide time to react to malicious proposals.
* **Design for worst-case behaviour.** Protocols must assume that any economic incentive will be abused. Simple donation-based weighting systems are vulnerable when they do not consider the source of funds.
* **Implement on-chain monitoring.** Monitoring for sudden large deposits, unusual proposals and massive transfers can help detect attacks in real time and trigger emergency shutdown procedures.
