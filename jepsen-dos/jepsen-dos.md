# Jepsen Lessons On Distributed Consistency

- Notes from a Kyle Kingsbury (Jepsen) talk
- Why vendor guarantees are not enough
- How real failures break "strongly consistent" systems


---

## "Strongly consistent" is not always consistent

- Even "strongly consistent" systems can lose data
- Case studies: RabbitMQ, etcd / Raft, Elasticsearch
- Bugs show up only under real failures, not happy-path tests
- Do not take marketing or docs as proof of correctness


---

## Network partitions are normal, not rare

- Partitions and similar faults will happen in production
- During a partition you must pick: consistency or availability
- This is the CAP trade off in practice, not theory
- Design for your choice and test it, instead of hoping "it will be fine"


---

## Fault injection as a first class practice

- Regular tests miss most distributed systems bugs
- Inject failures: network splits, node crashes, clock skew, GC pauses
- After each failure, check: is the data still correct?
- Jepsen style tests plus linearizability checkers (like Knossos) help
- Treat this as core QA, not an optional extra


---

## Linearizability is hard to achieve

- Linearizability: operations appear to happen in a single global order
- In practice, it is very easy to break
- Example: RabbitMQ "lock" based on a message in a queue
- If the lock holder is partitioned or GC paused, message is requeued
- Two clients can think they hold the same exclusive lock


---

## Use proven coordination services when you need strong guarantees

- Locks, counters, and leader election are easy to get wrong
- A non linearizable design will fail in surprising ways
- When you need strict guarantees, prefer:
  - Well tested coordination systems (Zookeeper, etcd)
  - Or designs with explicit safeguards and verification


---

## Consensus does not automatically mean correct

- Using Raft or Paxos does not save you from implementation bugs
- Example: etcd / Raft returning stale reads after successful writes
- Algorithm was sound, implementation was not
- Conclusion: protocol correctness is necessary, but not sufficient
- Always verify the actual system you deploy


---

## Beware "successful" but non durable operations

- Some systems acknowledge a write, then later drop it
- This can happen during failover or split brain recovery
- Watching confirmed writes simply vanish is very possible
- Defenses:
  - Use quorum acknowledgments
  - Use durable logging and fsync
  - Consider app level checks (checksums, read backs)


---

## Ops and environment strongly affect consistency

- Cluster behavior is sensitive to configuration and runtime conditions
- Heartbeat timeouts and failure detector settings matter a lot
- Overly aggressive timeouts can kill healthy nodes
- Long GC pauses can look like partitions and trigger split brain
- For stronger consistency:
  - Prefer conservative timeouts
  - Test settings under realistic load and latency


---

## Build a culture of empirical verification

- Do not blindly trust vendor claims about consistency or availability
- "Trust but verify" with:
  - Jepsen style tests
  - Chaos engineering and failure drills
- Goal: make sure real behavior matches expectations
- Do this before the system carries critical data or traffic


---

## Practical checklist

- Model your desired guarantees (linearizable? causal? eventual?)
- Make an explicit CAP choice for partitions
- Use proven components for hard problems (consensus, locks, leader election)
- Add fault injection and generative tests to CI
- Tune and test operational parameters under stress
- Continuously re verify after upgrades or config changes


---

## Further reading and tools

- Jepsen analyses and writeups: https://aphyr.com
- Jepsen project and tools (including Knossos): https://jepsen.io
- Case study writeups and blog posts:
  - https://devblogs.sh
  - https://highscalability.com
