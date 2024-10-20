# Eventual Consistency and Conflict Resolution - Part 1
## Why eventual consistency?
Replication is a common technique to provide higher availability and performance. However, as soon as we create replicas, we have to deal with the inconsistencies between them. An old idea to keep replicas consistent is to use a leader-follower scheme, i.e., one of the replicas becomes the leader and all writes go to it. The other replicas later receive updates from the leader and update their state. With this approach, once the leader replica fails, the service is unavailable until we pick a new leader using leader election algorithms such as Paxos or Raft. When we want to have strong consistency (a.k.a., linearizability) the failure of the leader is not the only case where our service becomes unavailable. Strong consistency requires replicas to talk to each other to serve any request. Now, if due to a network partition, replicas are disconnected and cannot talk to each other, our system becomes unavailable. This is known as the CAP theorem in the community [1] which simply says in presence of network partitions preventing replicas from talking to each other, you can pick either availability or strong consistency.

In addition to availability issues, strong consistency causes performance issues as well. Each write is more expensive, especially with cross-datacenter replication, as it needs the acknowledgment of multiple replicas. Moreover, we will have large latency for clients having large network delays to the leader. For example, when we use geo-replication, we create replicas in different geographical locations. With this leader-follower approach, all clients, no matter where they are, must talk to the leader to write, and this leader may be located in a remote datacenter.

As you see, we have many problems with strong consistency. At some point, the large internet companies with massive scales, dealing with customers from all around the world, started to say "You know what? let's give up on strong consistency, and let clients be able to write to any replica freely. This way, we can keep our services available all the time, achieve higher throughput, and earn more money!". Fortunately, for many applications, clients are tolerant to inconsistencies, i.e., they don't care if things are inconsistent for a short time. For some use cases, yes, we may have some problems. For example, due to eventual consistency, a shopping website may sell a single item to two buyers. For these cases, however, we usually can have "business solutions". For example, we can ask the supplier to restock the item. It may delay the delivery, or in the worst case, the restock may not be possible. In these cases, we can apologize and even compensate the customer for the inconvenience. Today, apologies and compensating customers are very common, even for things like hotel or flight bookings. The thing is, by using eventual consistency, many businesses earn way more money than the amount that they lose to compensate customers due to inconsistencies.



## Source
https://www.mydistributed.systems/2022/02/eventual-consistency-part-1.html