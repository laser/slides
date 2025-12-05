# Slide 1

## Notes from a Kyle Kingsbury (Jepsen) talk

The core ideas here come from Jepsen style analyses, which are based on
actually breaking real systems in realistic failure scenarios. Kingsbury takes
production grade databases and queues, runs them under nasty conditions, and
then checks whether they really uphold the guarantees they claim. Your notes
are a distilled summary of those lessons, not abstract theory or marketing
copy.

## Why vendor guarantees are not enough

Vendors often advertise "strong consistency" or "no data loss" but those claims
may assume ideal networks, no bugs, and perfect operations. In practice, real
deployments see weird failures, misconfigurations, and version specific bugs
that can invalidate the marketing guarantees. The point is that you should
treat vendor claims as hypotheses that you test, not as facts you build your
system around blindly.

## How real failures break "strongly consistent" systems

The talk walks through concrete examples where systems that were supposed to be
strongly consistent still lost data, returned stale reads, or violated ordering
guarantees. These failures usually show up only when you combine multiple
faults like network partitions, GC pauses, and restarts. The message is that
"strong consistency" on the label does not mean your data is actually safe
under real production failures.

---

# Slide 2

## Even "strongly consistent" systems can lose data

This means that the guarantees in the documentation do not magically protect
you from implementation bugs, configuration pitfalls, or interactions with your
environment. Even when a system aims for linearizability, there might be corner
cases it did not handle correctly, especially during failover or recovery. You
should assume that any complex distributed system has unknown failure modes
until you have tested it yourself.

## Case studies: RabbitMQ, etcd / Raft, Elasticsearch

The talk uses specific systems as examples: RabbitMQ for messaging, etcd (based
on the Raft consensus algorithm), and Elasticsearch as a distributed search and
storage engine. In each case, Jepsen tests uncovered surprising behaviors like
lost acknowledged messages or inconsistent reads. These examples are not to
shame those projects, but to show that even serious, widely used software can
have deep consistency bugs.

## Bugs show up only under real failures, not happy-path tests

If you only run unit tests and normal integration tests where the network is
stable and nodes do not crash, you are unlikely to see these bugs. The nastiest
consistency issues show up when you inject partial failures: a node goes down
mid write, messages are delayed, or the cluster is split. Therefore, you must
make failure testing part of your normal development and QA process, not an
afterthought.

## Do not take marketing or docs as proof of correctness

Documentation and blog posts are useful for understanding what a system is
intended to do, but they are not a formal proof that the system behaves
correctly in all conditions. There is usually a gap between the intended model
and reality, due to bugs, misconfigurations, or undefined edge cases. The only
reliable way to be confident is to measure and test the behavior of the actual
version you are running.

---

# Slide 3

## Partitions and similar faults will happen in production

Networks are not perfectly reliable: links break, switches misbehave, routes
flap, and firewalls are misconfigured. Over any nontrivial time horizon, you
will see lost packets, long delays, or complete connectivity loss between parts
of your cluster. Treating partitions as a rare edge case is dangerous; you
should assume they will happen and design your system to behave predictably
when they do.

## During a partition you must pick: consistency or availability

The CAP theorem says that in the presence of a partition, you cannot have both
strong consistency and full availability. In practical terms, if nodes cannot
talk to each other, either some part of the system must refuse operations
(sacrificing availability) or you allow operations that might diverge from each
other (sacrificing consistency). Your architecture needs to make this trade off
explicit instead of leaving it to chance.

## This is the CAP trade off in practice, not theory

People often talk about CAP in abstract terms, but the talk shows real bugs
that arise when systems do not have a clear CAP strategy. For example, allowing
both sides of a partition to accept writes can lead to split brain and data
loss when the partition heals. Seeing CAP as a practical design constraint
forces you to spell out exactly what should happen when connectivity is
partially lost.

## Design for your choice and test it, instead of hoping "it will be fine"

Once you decide whether you prefer consistency or availability under
partitions, you need to reflect that in code paths, timeouts, quorum rules, and
operational procedures. Then you should simulate partitions to verify that the
system behaves according to that choice. The worst situation is a system that
"accidentally" behaves in some inconsistent way because no one thought through
the partition behavior.

---

# Slide 4

## Regular tests miss most distributed systems bugs

Typical test suites assume the network and machines are healthy and focus on
functional correctness of business logic. They might never exercise scenarios
where messages are delayed, reordered, or dropped, or where leaders change
rapidly. As a result, the subtle concurrency and consistency bugs that only
appear under stress or partial failure remain hidden until production.

## Inject failures: network splits, node crashes, clock skew, GC pauses

Fault injection means deliberately causing bad conditions, like cutting off
network links between subsets of nodes or killing a process at a specific
point. You can also simulate clock skew to see what happens when nodes disagree
about time, or force long GC pauses that halt a process unexpectedly. By doing
this in a controlled environment, you can see how the system handles conditions
that are painful but realistic.

## After each failure, check: is the data still correct?

It is not enough to see that the cluster "recovers" or that services restart;
you must check whether the data satisfies your consistency model after the
failure. That usually means comparing the observed history of operations
against a formal model such as linearizability or serializability. Many systems
will keep running but silently return wrong results or lose updates, which you
only catch by checking data correctness explicitly.

## Jepsen style tests plus linearizability checkers (like Knossos) help

Jepsen style testing frameworks generate random client operations, inject
failures on the cluster, and log all events. A checker like Knossos then tries
to reconstruct a legal history under a given consistency model; if it fails,
you know the system violated that model. This gives you a more rigorous signal
than eyeballing logs or relying on ad hoc application checks.

## Treat this as core QA, not an optional extra

For systems that store important data or coordinate critical processes, failure
testing is as essential as unit tests or code review. If you do not budget time
for it, you are effectively accepting unknown data loss or inconsistency risks.
Treating it as a first class responsibility helps justify running these tests
regularly and integrating them into CI or pre release validation.

---

# Slide 5

## Linearizability: operations appear to happen in a single global order

Linearizability is a strong consistency property that says every operation
appears to take effect at some instant between its start and end time, and all
clients see results consistent with a single global sequence of operations. It
is stronger than simple eventual consistency, because it constrains how reads
and writes can interleave. Many APIs that use words like "atomic" or "strongly
consistent" are trying to offer something close to this.

## In practice, it is very easy to break

Achieving linearizability requires careful handling of leader election,
replication, timeouts, and failure detection. Small mistakes, like failing to
wait for a quorum or letting a stale leader continue to serve, can admit
histories that violate the property. Under normal, healthy conditions the
system might appear fine, but under partitions or restarts subtle ordering
violations can show up.

## Example: RabbitMQ "lock" based on a message in a queue

RabbitMQ tried to implement a distributed lock by having a client consume a
special message from a queue; while it holds that message, it is considered to
own the lock. This seems simple, but the queue semantics and failure handling
mean that, under certain conditions, that message can be given to more than one
consumer over time. The talk uses this as a concrete illustration of how naive
lock designs can go wrong.

## If the lock holder is partitioned or GC paused, message is requeued

When the node that holds the lock stops acknowledging messages because it is
partitioned from the cluster or stuck in GC, RabbitMQ assumes it is dead or
unresponsive. The broker then requeues the lock message so another consumer can
pick it up. That logic is fine for normal work queues, but it breaks the
assumption that only one client can ever hold the lock at once.

## Two clients can think they hold the same exclusive lock

Because the system can reassign the lock message while the original holder
might still be alive but temporarily stalled, you can end up with two clients
both believing they own the lock. Each client might then perform "exclusive"
operations concurrently, corrupting shared state. This demonstrates how easily
a design that seems correct informally can violate linearizability when faults
occur.

---

# Slide 6

## Locks, counters, and leader election are easy to get wrong

These distributed primitives seem simple conceptually, but they depend on
subtle guarantees about message ordering and failure handling. Naive
implementations often ignore corner cases like delayed messages or reappearing
nodes, which can lead to duplicate leaders, lost increments, or broken mutual
exclusion. Because these bugs are rare and hard to reproduce, they can hide for
a long time.

## A non linearizable design will fail in surprising ways

If your design does not guarantee linearizability, clients can see histories
where operations appear to overlap or reorder in unexpected ways. For something
like a lock or a counter, that can mean two writers both think they are safe to
proceed, or a count that skips or repeats values. These anomalies often
manifest only under high load or partial failure, which makes them especially
nasty in production.

## When you need strict guarantees, prefer:

This is a reminder that you do not always need the strongest possible
consistency, but when you do, you should be careful about how you get it. For
critical metadata such as locks, membership, or leader election, it is often
better to rely on components that are designed and tested specifically for that
purpose. Reinventing these mechanisms is rarely worth the risk.

- **Well tested coordination systems (Zookeeper, etcd):** Systems like
  Zookeeper and etcd are built around consensus protocols and have had years of
  scrutiny, bug fixes, and Jepsen style testing. They provide APIs for things
  like ephemeral nodes, leader election, and key value storage with well
  defined consistency guarantees. Using these systems as the source of truth
  for coordination can be safer than building ad hoc solutions into each
  service.

- **Or designs with explicit safeguards and verification:** If you do build
  your own coordination mechanisms, you should back them with formal reasoning
  and tests rather than trusting intuition. This might mean specifying the
  expected behavior under failure, using model checking, and running Jepsen
  style tests against your implementation. The idea is to make your guarantees
  explicit and tested, not implicit and assumed.

---

# Slide 7

## Using Raft or Paxos does not save you from implementation bugs

Raft and Paxos are consensus algorithms that, when implemented correctly,
provide strong guarantees about log replication and leader election. However,
real systems embed these algorithms in large codebases with many other moving
parts. Any bug in the implementation, integration, or configuration can break
the assumptions of the protocol, so just saying "we use Raft" is not enough to
be safe.

## Example: etcd / Raft returning stale reads after successful writes

In one Jepsen analysis, etcd under certain conditions returned stale data to
clients even after a write was reported as successful. That meant the system
violated the linearizability it aimed to provide, despite using Raft
underneath. The issue came from how reads were handled relative to leader
election and log commitment, not from the Raft paper itself.

## Algorithm was sound, implementation was not

The Raft algorithm as described in the literature is proven to meet certain
safety properties, but those proofs assume a correct implementation and certain
conditions. When the code deviates from the algorithm, adds optimizations, or
interacts with other components, new bugs can appear. This distinction between
algorithm and implementation is crucial: you deploy code, not the paper.

## Conclusion: protocol correctness is necessary, but not sufficient

It is still valuable to base your system on a well studied protocol, because
that gives you a solid foundation. But you must also consider practical
concerns like timeouts, logging, replay, and metrics, which are outside the
protocol spec. The talk emphasizes that you need both: a correct protocol and a
robust, tested implementation.

## Always verify the actual system you deploy

Different versions, builds, and configurations of the same product can behave
differently, especially as new features and optimizations are added. Before
trusting a system in production, it is wise to run your own tests against the
exact version and configuration you plan to use. This helps catch regressions
or environment specific bugs that would not show up in someone else's test
results.

---

# Slide 8

## Some systems acknowledge a write, then later drop it

This refers to scenarios where the client receives a "success" response for a
write, but due to a crash, failover, or replay issue, that write never makes it
into the durable state of the cluster. From the client's perspective, the
operation succeeded, but when they later read, the data is missing. This "lost
acknowledged write" is one of the most serious kinds of consistency failure.

## This can happen during failover or split brain recovery

Common triggers include leader changes when the current leader has accepted
writes that are not fully replicated, or split brain conditions where two
leaders accept conflicting writes. When the system resolves the conflict or
rolls back to a previous state, some of those writes can be discarded. If the
system has already told clients those writes are durable, that is a violation
of their expectations.

## Watching confirmed writes simply vanish is very possible

Jepsen tests have shown real systems losing hundreds or thousands of confirmed
operations during aggressive failure scenarios. From the outside, it looks like
the system is working fine until you perform a consistency check and discover
missing data. This is why relying only on "no errors in logs" or "cluster looks
healthy" is not enough to guarantee safety.

## Defenses:

There are practical measures you can take to reduce the risk of losing
acknowledged writes. None of them are free, but together they can make your
system more robust against partial failures and reconfigurations. They focus on
ensuring writes are replicated and persisted before you tell the client they
have succeeded.

- **Use quorum acknowledgments:** Quorum acknowledgments require that a write
  be accepted by a majority (or some configured subset) of nodes before the
  system reports success to the client. This means that even if one node fails
  after acknowledging, the data still exists on other nodes in the quorum. It
  lowers the chance that a single node failure will cause the loss of a
  confirmed write.

- **Use durable logging and fsync:** Durable logging means persisting each
  write to a log on disk, typically with fsync or an equivalent mechanism,
  before acknowledging it to the client. This ensures that even if a node
  crashes and restarts, it can recover the acknowledged writes from its log.
  Skipping or batching fsyncs can improve performance but increases the risk of
  losing recent writes during crashes.

- **Consider app level checks (checksums, read backs):** At the application
  level, you can add extra confirmation that data is really stored as expected.
  For example, after writing, you might perform a read to verify the value, or
  store checksums and periodically validate them. These strategies cannot fix a
  broken storage layer, but they can help you detect data loss earlier and give
  you evidence when something has gone wrong.

---

# Slide 9

## Cluster behavior is sensitive to configuration and runtime conditions

Even if the code is correct, the way you configure timeouts, retries, and
resource limits can strongly influence how the cluster behaves under load.
Small changes in these parameters can shift when nodes decide others have
failed, or how quickly they retry operations. Runtime factors like CPU
saturation and disk latency also affect how the system reacts to stress.

## Heartbeat timeouts and failure detector settings matter a lot

Failure detectors use heartbeats or similar signals to decide when a node is
unresponsive and should be considered dead. If the timeout is too short,
transient delays or GC pauses can look like permanent failures; if it is too
long, real failures take too long to detect. Tuning these values requires
understanding your network latency distribution and workload characteristics.

## Overly aggressive timeouts can kill healthy nodes

When timeouts are too aggressive, nodes may frequently decide that their peers
have failed even when they are just slow. This can cause unnecessary leader
elections, data movement, and partition like behavior, stressing the system
further. In extreme cases, you can get oscillations where nodes repeatedly drop
and rejoin, making the cluster unstable.

## Long GC pauses can look like partitions and trigger split brain

Languages with stop the world garbage collectors can pause a process long
enough that it misses heartbeats or fails to respond to peers. Other nodes then
interpret this as a failure or partition and may elect a new leader or start
serving conflicting data. When the paused node resumes, it may believe it is
still leader or has up to date state, which can lead to inconsistencies.

## For stronger consistency:

There are concrete operational recommendations for teams that value consistency
over maximum availability or latency. You can trade some performance and fast
failover for more predictable, safe behavior. The main idea is to choose
conservative settings and validate them experimentally.

- **Prefer conservative timeouts:** Choosing timeouts that are long enough to
  tolerate typical jitter and GC pauses reduces the chance of false failure
  detections. This may mean slower reaction to real failures, but it also
  reduces needless leader churn and split brain risk. Conservative settings are
  especially important in multi region deployments with higher latency
  variance.

- **Test settings under realistic load and latency:** Configuration that looks
  fine in a small, idle test cluster may behave very differently under real
  production load and network conditions. You should test your settings in an
  environment that resembles production in terms of traffic patterns, latency,
  and resource usage. Running chaos tests or stress tests while monitoring
  behavior gives you confidence that the configuration is robust.

---

# Slide 10

## Do not blindly trust vendor claims about consistency or availability

Vendors have incentives to present their systems in the best possible light and
may gloss over failure scenarios or edge cases. Marketing terms like "zero data
loss" or "linearly scalable" often hide important assumptions and trade offs.
As an engineer, you need to interpret these claims with skepticism and probe
their limits.

## "Trust but verify" with:

Start by assuming the system might be correct, but then actively look for
evidence. Instead of arguing about guarantees on paper, you run experiments
that mimic your actual workload and failure scenarios. The results inform
whether and how you can safely use the system.

- **Jepsen style tests:** Using Jepsen style tests means generating sequences
  of operations, injecting failures, and checking the resulting histories
  against your desired consistency model. These tests are designed to be
  adversarial: they try to break the system rather than just confirm it works.
  Passing such tests does not prove correctness forever, but it gives you far
  more confidence than static claims.

- **Chaos engineering and failure drills:** Chaos engineering focuses on
  injecting failures into live or realistic systems to observe how they behave
  and recover. This might involve killing processes, disabling network links,
  or degrading dependencies on purpose. Failure drills, like game days, help
  teams practice responding to these events, improving both the system and the
  humans operating it.

## Goal: make sure real behavior matches expectations

The core objective is alignment between the mental model you design around and
what the system actually does under pressure. If you think your database
provides linearizable writes, you want to see that supported by empirical
evidence. When behavior diverges from expectations, you either adjust the
system or update your mental model.

## Do this before the system carries critical data or traffic

Running these tests early avoids discovering fundamental problems only after
users rely on the system in production. Once you put critical workloads on top,
changing components or behavior becomes much more expensive. By front loading
this verification, you can choose technologies and configurations that are
appropriately safe for the intended use.

---

# Slide 11

## Model your desired guarantees (linearizable? causal? eventual?)

Before picking tools, you should decide what level of consistency your
application actually needs. Some features truly require linearizable semantics,
while others can tolerate eventual consistency or weaker models like causal
consistency. Writing down these requirements helps you choose or design systems
that are a good fit instead of assuming one size fits all.

## Make an explicit CAP choice for partitions

For each key system or feature, decide whether, during a partition, you prefer
to reject operations to preserve consistency, or accept operations and
reconcile later. Document these decisions and propagate them into design and
configuration choices. Being explicit avoids accidental behavior where
different components make conflicting CAP trade offs.

## Use proven components for hard problems (consensus, locks, leader election)

Do not try to hand roll your own Raft or Paxos or ad hoc locking scheme unless
you have very strong reasons and expertise. It is usually better to build on
top of well tested libraries or external services that already address these
problems. This lets you focus on application logic rather than reproducing
complex consensus algorithms.

## Add fault injection and generative tests to CI

Integrating failure tests into continuous integration means you regularly
exercise your system under bad conditions, not just before big releases.
Generative tests create varied, randomized operation sequences that reveal
corner cases you might not think to script by hand. As you fix bugs, these
tests can prevent regressions by replaying previous failure scenarios.

## Tune and test operational parameters under stress

Configuration tuning is not a one time activity; you should revisit it as your
workload and environment evolve. When you change things like timeouts,
concurrency limits, or GC settings, verify the impact with stress tests and
monitoring. This helps ensure that performance improvements do not come at the
cost of hidden consistency risks.

## Continuously re verify after upgrades or config changes

New versions of software introduce code changes that may interact differently
with your environment and configuration. Similarly, infrastructure changes like
new network topologies or storage systems can affect behavior. Periodically
rerunning your verification suite after such changes helps catch new issues
early and maintains confidence in your guarantees over time.

---

# Slide 12

## Resources

- **Jepsen analyses and writeups:** <https://aphyr.com> - The aphyr.com site
  hosts detailed Jepsen reports on many databases, queues, and coordination
  systems. Each writeup walks through the tests performed, the observed
  anomalies, and how the vendors responded. Reading these reports can help you
  understand common failure patterns and how different systems behave under
  stress.

- **Jepsen project and tools (including Knossos):** <https://jepsen.io> -
  Jepsen.io focuses on the tooling and methodology that support Jepsen style
  testing. It includes libraries for building test harnesses, injecting faults,
  and checking histories with tools like Knossos. If you want to run similar
  tests against your own systems, this is where you start.

- **Case study writeups and blog posts:** These case studies often summarize
  complex incidents or testing campaigns in more narrative form. They are
  useful for learning how theoretical concepts translate into real world
  outages and bugs.
  - <https://devblogs.sh> - Articles that expand on Jepsen findings and related
    distributed systems topics. Posts there can provide deeper technical
    explanations, code snippets, or follow up experiments.
  - <https://highscalability.com> - Essays and postmortems about building and
    running large scale systems. Articles there often discuss trade offs
    between consistency, availability, performance, and cost, sometimes
    referencing Jepsen or similar testing approaches.
