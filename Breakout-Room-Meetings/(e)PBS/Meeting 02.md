# ePBS breakout #2 notes

## Details
June 7 (Fri), 2024 , 14:00 UTC 🕑 ❗

Duration: 60 minutes

[Zoom link](https://ethereumfoundation.zoom.us/j/87569210985?pwd=3Cv1hDh7If4cq9IMXvNln1CtqQ72MR.1#success)

[Agenda](https://github.com/ethereum/pm/issues/1060)

### **Notes originally documented [here](https://hackmd.io/@ttsao/epbs-breakout2)**

## Agenda

* EIP Status

* Spec discussion

* Implementation complexity discussion

## Action items:

* The consensus layer client should check if it can process a consensus block in under 1 second. This includes the entire end-to-end process, similar to a pre-merge setting.
* The consensus layer client should provide a beacon API to retrieve the fork choice weight of a specific block.
* The fork choice specification should include a backoff scheme to recover from prolonged network liveness degradation.
* The fork choice specification would benefit from more reviewers to examine competitive edge cases.
* The validator specification should move the sync committee duty from 4 seconds to 3 seconds into the slot.
* The consensus specification requires more analysis on whether PTC slashing is fully necessary and if the attack vector of spamming multiple PTC statuses is a concern.
* The consensus specification may need to move the builder reveal cutoff and PTC attestation cutoff to an earlier time to mitigate the builder reveal timing game.

## Summary:

* We proposed to start the meeting with some implementation details and current concerns.
* The first concern is the timing of block processing.
* Today, everything (consensus, execution, blobs) happens in the hot path within 2-4 seconds.
* In ePBS, the hot path is pipelined across the slot. We assume a consensus client can process a block in under 1 second, which we believe is feasible.
* Builders should release the block as soon as there is sufficient weight. The consensus layer client should provide a beacon API to return the fork choice weight of a block root. We shouldn’t expect builders to figure this out themselves.
* PTC votes on the timeliness and availability of the payload, not its execution validity. Execution validation can be deferred until the beacon attestation deadline of the next slot, which is a significant improvement.
* We discussed the complications of withdrawals, given the concept of empty slots in ePBS. If there are dependencies between consensus and execution, like withdrawals, they need careful design. Currently, withdrawals originate from the consensus layer and finish at the execution layer. If there is an empty slot, the withdrawal must be remembered and fulfilled in the next slot. To reduce complexity, we should minimize interdependencies.
* We clarified that there will be two post states: one for consensus and one for execution. The current fork choice specification reflects this.
* We talked about local building, which follows the same process as today. Geth could return the bid or the full execution block, and the execution block could be gossiped early since both the proposer and builder are trusted.
* We agreed that we need to conduct a fork choice liveness analysis to account for performance under poor network conditions. If chain growth stops completely when block latency exceeds SECONDS_PER_SLOT / 3, we could implement a backoff scheme similar to the one Francesco originally designed for Peerdas.
* We are looking for more people to review fork choice, as it is the most critical part of the ePBS change. More reviews on competitive edge cases will be helpful.
* We confirmed that no team has started implementation except for Prysm.
* We discussed the current ePBS block auction versus forward-compatible designs like slot auctions and ticket auctions. Since slot and ticket auctions don’t have a Python spec, it’s hard to analyze them. We think slot auctions are relatively easy if we can prove that builder double proposing won’t cause much damage. There hasn’t been much progress until someone seriously examines slot auctions under current LMD ghost and FFG.
* We mentioned the need to change the sync committee deadline from 4 seconds to 3 seconds. We talked about whether the current sync committee could be used for PTC duty. It’s difficult because of aggregation, and the sync committee is long-lived rather than shuffled every slot.
* We discussed whether to add slashing for PTC. If PTC members gossip all possible outcomes, there’s no hard penalty. If PTC is not included in the block, it loses the reward. We need more analysis on how this might negatively affect the network.
* We talked about the builder timing game. It could be an issue, so we might move the builder reveal time and PTC attestation cutoff time earlier. This seems feasible since attestations arrive in under a second, and builders are well connected.
* We discussed PTC rewards. Currently, PTC members are borrowed from the beacon committee, a very small subset (521 out of 30,000). PTC members get rewards for sending correct attestations and are penalized for incorrect ones. They are welcome to send beacon attestations, which will be counted but not rewarded. This approach simplifies specification and implementation. PTC doesn’t add LMD weight; it’s only for the next slot attestation to apply reveal boost.
* Finally, We were asked, "What incentive do current builders and proposers have to switch to ePBS?" This is an important question, but we didn’t have time to cover it. We decided to discuss this in the next call.
