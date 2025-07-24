# Beanstalk Governance Attack (April 2022)

## Exploit overview

In April 2022, the Beanstalk stablecoin protocol was drained of roughly $181 million in collateral. Attackers exploited the protocol’s decentralized governance system by pushing through two malicious proposals that siphoned funds from the treasury. Leveraging a massive flash loan, the attacker temporarily acquired a supermajority of governance tokens, executed an emergency commit, and transferred the assets to their own wallet address 0x1c5dCdd006EA78a7E4783f9e6021C32935a10fb4 and a Ukraine relief donation address. Although the protocol lost $181 million, the attacker ultimately netted around $76 million after repaying the flash loans.

## Root cause

The core vulnerability stemmed from Beanstalk’s governance mechanism. Voting power was directly proportional to the amount of governance tokens deposited into the protocol’s Diamond contract. The protocol included an `emergencyCommit` function, allowing proposals to be enacted with a 2/3 supermajority after a 24-hour waiting period. Critically, there were no safeguards against flash-loaned token deposits, enabling the attacker to borrow tokens, temporarily inflate voting power, and push through proposals. The lack of controls, combined with unaudited emergency functions, made the attack possible.

## Attack path

1. Deploy malicious proposals: The attacker created two proposals (Proposal #18 and #19) that appeared innocuous but included logic to transfer protocol funds to 0x1c5dCdd006EA78a7E4783f9e6021C32935a10fb4 and the Ukraine donation wallet.

2. Wait for timelock: After submission, the 24-hour governance timelock period elapsed.

3. Flash-loan and deposit: The attacker used flash loans (reportedly over $1 billion) to acquire governance tokens and deposit them into the Diamond contract, gaining around 79% voting power—well above the 66.7% threshold.

4. Execute proposals: The attacker invoked emergencyCommit to execute the proposals, draining the protocol’s reserves.

5. Repay and launder: After transfers to 0x1c5dCdd006EA78a7E4783f9e6021C32935a10fb4 and the donation address, the attacker repaid the loans and routed most funds through Tornado Cash, keeping approximately 24,930 ETH (~$76 million).

## On-chain indicators

Several on-chain signals preceded and accompanied the attack:

* **Large governance deposits:** Immediately prior to the `emergencyCommit` call, a single transaction deposited an unusually large amount of governance tokens into the Diamond contract, pushing the attacker's voting share above the super-majority threshold.
* **Multiple malicious proposals:** Two proposals were introduced in quick succession with similar payloads. Monitoring unusual proposal contents could have raised alarms.
* **Sudden value transfer:** A single execution transferred hundreds of millions of dollars worth of tokens from Beanstalk to addresses not previously associated with the protocol.
* **Flash-loan repayment and mixing:** Within the same block, the attacker repaid the flash loan and routed remaining funds to Tornado Cash.

## Lessons learned

The Beanstalk incident highlights several important security lessons for DeFi projects:

* **Limit instantaneous voting power.** Governance systems should restrict how much influence can be obtained in a single transaction or block. Time-weighted voting or blocking flash-loaned deposits can mitigate similar attacks.
* **Audit and monitor governance functions.** Critical functions such as `emergencyCommit` should be thoroughly audited and monitored. Long timelocks and off-chain alerts can provide time to react to malicious proposals.
* **Design for worst-case behaviour.** Protocols must assume that any economic incentive will be abused. Simple donation-based weighting systems are vulnerable when they do not consider the source of funds.
* **Implement on-chain monitoring.** Monitoring for sudden large deposits, unusual proposals and massive transfers can help detect attacks in real time and trigger emergency shutdown procedures.
