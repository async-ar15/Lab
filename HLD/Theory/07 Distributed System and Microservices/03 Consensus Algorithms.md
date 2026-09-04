Sahil: Hi Sourajit! Let’s play something.

Sourajit: Let’s play football.

Sahil: Yeah sure! We both love to play that.

Both Sahil and Sourajit wanted to play a game and Sourajit suggested playing football. Sahil agreed with Sourajit’s proposal. So, they agreed upon a common value suggested by one of them and took action on that. This agreement on common value is known as consensus.

What is Consensus in Distributed System?
In a distributed system, multiple computers (known as nodes) are mutually connected with each other and collaborate with each other through message passing. Now, during computation, they need to agree upon a common value to coordinate among multiple processes. This phenomenon is known as Distributed Consensus.

Press enter or click to view image in full size

Source: https://www.preethikasireddy.com/post/lets-take-a-crack-at-understanding-distributed-consensus
Press enter or click to view image in full size

Source: https://www.preethikasireddy.com/post/lets-take-a-crack-at-understanding-distributed-consensus
Press enter or click to view image in full size

Source: https://www.preethikasireddy.com/post/lets-take-a-crack-at-understanding-distributed-consensus
Press enter or click to view image in full size

Source: https://www.preethikasireddy.com/post/lets-take-a-crack-at-understanding-distributed-consensus
Why is Consensus required?
In a distributed system, it may happen that multiple nodes are processing large computations distributedly and they need to know the results of each node to keep them updated about the whole system. In such a situation, the nodes need to agree upon a common value. This is where the requirement for consensus comes into the picture.

How to achieve Consensus in Distributed System
Let’s consider a distributed system, where n nodes are connected with each other. one node sends a message to all other nodes saying “Guys! I chose value v do you all agree?”.

Now, two scenarios can come up.

All agreed-upon values v
Some of them disagree
Case 1: All the nodes can carry out their task keeping v as the value in their mind.

Case 2: Suppose one of the nodes suggests “I prefer value w”. In this type of situation achieving consensus is complicated.

To achieve consensus all the nodes in distributed system should follow the same protocol when communicating. There are three basic conditions that need to be satisfied by the system for consensus algorithms.

Agreement: All non-faulty nodes should agree on the same value.
Validity: If a system has decided on a value v then that value should be suggested by one of the non-faulty nodes of the system and all other non-faulty nodes must decide that value v.
Termination: Every non-faulty node should agree upon some value. If one of the non-faulty nodes agrees upon value v, consensus can not be achieved.
Non-faulty node means, node which is not crashed or attacked or malfunctioning.

Challenges in Distributed Consensus
A distributed system can face mainly two types of failure.

Crash failure
Byzantine failure
Crash failure occurs when a node is not responding to other nodes of the system due to some hardware or software or network fault. This is a very common issue in distributed systems and it can be handled easily by simply ignoring the node’s existence.

Press enter or click to view image in full size

Crash Failure
Byzantine failure is a situation where one or more node is not crashed but behaves abnormally and forward a different message to different peers, due to an internal or external attack on that node. Handling this kind of situation is complicated in the distributed system.

Press enter or click to view image in full size

Byzantine Failure: faulty node sending different messages to different peers
A consensus algorithm, if it can handle Byzantine failure can handle any type of consensus problem in a distributed system.

Consensus Algorithms
Voting-based Consensus Algorithms
Some of the first implementations of consensus algorithms began to employ various voting-based techniques. These have enough fault tolerance and sufficient mathematical proof to assure security and stability. However, because of their democratic character, these algorithms are extremely slow and inefficient, especially as the network becomes larger.

Practical Byzantine Fault Tolerance
Let’s understand Practical Byzantine Fault Tolerance (pBFT) through an example.

Become a Medium member
Imagine that several divisions of the Byzantine army are camped outside an enemy city, each division commanded by its own general.

The generals can communicate with one another only by messenger. After observing the enemy, they must decide upon a common plan of action.

However, some of the generals may be traitors, trying to prevent the loyal generals from reaching an agreement. The generals must decide on when to attack the city, but they need a strong majority of their army to attack at the same time.

The generals must have an algorithm to guarantee that (a) all loyal generals decide upon the same plan of action, and (b) a small number of traitors cannot cause the loyal generals to adopt a bad plan.

The loyal generals will all do what the algorithm says they should, but the traitors may do anything they wish. The algorithm must guarantee condition (a) regardless of what the traitors do. The loyal generals should not only reach an agreement but should agree upon a reasonable plan.

Father of Distributed Systems Leslie Lamport proved that —

If more than two-thirds of all nodes in a system are honest then consensus can be reached.

This algorithm works on the above-mentioned principle. The distributed system is divided into three phases (pre-prepare, prepare, commit) and nodes are sequentially ordered with one node being the Primary node (or leader node) and others as the Secondary node (or backup node). The objective is that all non-faulty nodes help in achieving a consensus regarding the state of the system using the majority rule.

pBFT consensus rounds are —

The client sends a request to the primary node.
The primary nodes broadcast the request to all secondary nodes.
All the nodes perform the service that is requested and send it to the client as a reply.
The request is served successfully when the client received a similar message from at least two-thirds of the total nodes.
The primary node is replaced in every consecutive round using the view change protocol if a predetermined amount of time passes without the leading node broadcasting a request to the backup (secondary) nodes.

2. Other Notable Algorithms

There are other voting-based consensus algorithms like —

HotStuff
Paxos
Raft etc…
Proof-based Consensus Algorithms
With the development of blockchain technology and distributed ledgers, networks became considerably broader and permissionless. For these circumstances, a proof-based consensus technique seemed preferable. In this case, a participant must show adequate proof of something in order to contribute to decision-making.

There are several Proof-based Consensus algorithms —

Proof of Work (PoW)
Proof of Stake (PoS) etc…
Application of Distributed Consensus
Consensus algorithms are used in many real-world applications in distributed or decentralized networks —

✅ Blockchain and cryptocurrencies

✅ Google page-rank

✅ Load balancing etc…