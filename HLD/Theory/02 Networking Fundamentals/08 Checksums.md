Data does not always arrive exactly the way it was sent. A network packet can get damaged, a disk can return an old block, or a download can stop halfway through. The tricky part is that the bytes usually do not announce, "I am broken."

A checksum helps catch this. It is a small value calculated from a larger piece of data. Later, another part of the system calculates the value again and compares the two results.

Checksums show up everywhere in real systems: network frames, TCP and UDP packets, storage blocks, database pages, backups, container images, and replication logs.

The important design question is: what kind of problem are you trying to catch? A CRC is good for accidental damage. A cryptographic hash is better when you need a strong content fingerprint. An HMAC or digital signature is used when you also need to know who produced the data.

This chapter explains how checksums work, where they are used, and how to choose the right kind for a system.


1. What a Checksum Is
A checksum is a short value calculated from a larger piece of data.

For example, a storage system may store:

Plaintext







123
Later, when the object is read back, the system calculates crc32c again over the bytes it received. If the new value is different, the system knows the object changed somehow. It may be corrupted, cut short, mixed with the wrong bytes, or damaged in another way.




Large Data
8 MB object

Checksum
Function
e.g. CRC-32C

Compact Value
0x7f9c2ba4

A checksum is useful because it is much smaller than the data. A 4-byte CRC can protect a network frame. A 32-byte SHA-256 digest can identify a file that is many gigabytes large.

Because the checksum is smaller than the data, two different inputs can sometimes produce the same checksum. This is called a collision. Good algorithms make accidental missed corruption extremely unlikely for the problems they are designed to catch.


2. What Checksums Can and Cannot Tell You
Checksums answer a narrow question:

Do these bytes match the bytes I expected, according to this algorithm?

They are useful for detecting:

Bit flips
Truncated writes
Partial reads
Wrong block reads
Corrupted network frames
Damaged files
Bad disk sectors
Memory or DMA corruption
Replication or backup transfer errors
They do not automatically answer:

Who created the data?
Was the checksum itself tampered with?
Is this data authorized?
Is the algorithm secure against an attacker?
Is the application-level operation correct?
If an attacker can change both the file and the checksum, a plain checksum does not protect you. The attacker can modify the file and then calculate a new checksum for the modified file.

When people may be trying to tamper with the data, use a stronger tool: HMACs, digital signatures, signed manifests, TLS, package signatures, or another trusted verification mechanism.


3. How Verification Works
The verification flow is the same in networks, files, databases, and object storage.




Yes

No

Write or send data

Compute checksum

Store data + checksum
or transmit both

Read or receive data

Recompute checksum

Match?

Accept data

Reject, retry, repair,
or alert

What happens after a mismatch depends on where the mismatch was found:

A network card may drop the frame and rely on a higher layer to retransmit.
TCP may discard a bad segment. The sender will send it again when it notices data is missing.
A storage engine may read another replica.
An object store may fail the request and return a data-integrity error.
A backup system may mark the backup as unusable.
A database may stop startup rather than replay a corrupt page.
Checksums detect corruption. They do not repair it by themselves. Recovery needs another mechanism, such as retransmission, another replica, backup restore, a repair job, or manual intervention.


4. Types of Integrity Checks
Not all checksums solve the same problem. Choose the algorithm based on what you are protecting against.

Parity
A parity bit records whether a group of bits contains an even or odd number of 1 bits.

Parity can detect any single-bit error in that group. It cannot detect many multi-bit errors. It is simple and cheap, but too weak for most application-level integrity checks.

Simple Additive Checksums
A simple checksum may add bytes or words together and store part of the result.

These are fast and easy to implement, but weak. They may miss reordered bytes or error patterns that cancel each other out.

Use simple additive checksums only when an existing protocol requires them or when the failure cases are very limited.

CRC
CRC (Cyclic Redundancy Check) algorithms are a family of fast checks used heavily in networking and storage. The math behind them is more involved than simple addition, but the practical idea is simple: CRCs are designed to catch common patterns of accidental damage.

CRCs are especially good at catching burst errors, bit flips, and transmission noise.

Common examples:

CRC-32
CRC-32C
CRC-64
CRC-32C is widely used in storage and networking because it catches common errors well and many CPUs can compute it quickly in hardware.

CRC is not a cryptographic algorithm. An attacker can deliberately modify data and compute a new CRC.

Cryptographic Hashes
A cryptographic hash produces a fixed-size digest, such as a SHA-256 value. It is designed so that attackers cannot realistically find another input with the same digest or work backward from the digest to the original input.

Common examples:

SHA-256
SHA-384
SHA-512
BLAKE3
Cryptographic hashes are useful for:

Verifying downloads
Identifying content by digest
Avoiding duplicate storage for immutable blobs
Building Merkle trees
Verifying container images and package files
Comparing large files without reading them repeatedly
MD5 and SHA-1 should not be used when security matters because attackers can create collisions for them in practice. You may still see them in older systems, but new designs should prefer SHA-256 or a modern alternative.

HMACs and Digital Signatures
If you need to prove that data came from a trusted party and was not modified, a plain checksum or hash is not enough.

Use an HMAC when both sides share a secret key. Use a digital signature when one party signs the data and many other parties need to verify it.

The difference is about who is allowed to verify and who is allowed to create valid proof.

An HMAC uses one shared secret key. Both the sender and receiver have the same key. The sender calculates a tag from the data and the key, then sends the tag with the data.

Because verifying an HMAC requires the same key used to create it, anyone who can verify an HMAC can also create one. That makes HMAC a good fit for two parties that already trust each other, such as a webhook provider and the backend receiving its calls.

An HMAC proves the data came from someone holding the shared secret. It does not prove exactly which holder created it.

A digital signature uses a key pair. The signer keeps a private key and signs the data. Everyone else verifies the signature with the matching public key. Verifiers cannot create signatures because they never see the private key.

That is what lets one publisher sign a software release and millions of people verify it safely. The cost is key management: verifiers must trust that the public key really belongs to the publisher.

Examples:

Signed package repositories
Signed container images
Webhook signatures
API request signatures
Software update manifests
HMACs and signatures prove two things at once: the data was not changed, and it came from someone trusted.


5. Algorithm Comparison
Mechanism	Good For	Not Good For
Parity	Very cheap single-bit error detection	Multi-bit corruption, files, deliberate tampering
Additive checksum	Simple protocol checks	Strong error detection or security
CRC-32 / CRC-32C	Accidental corruption in frames, blocks, files	Malicious tampering
SHA-256	Strong content digest and file verification	Proving who produced the digest
HMAC	Integrity and authenticity with a shared secret	Public verification without sharing the secret
Digital signature	Proving data came from a trusted signer	Very high-throughput per-packet checks
The common mistake is choosing a tool that solves a different problem. A CRC is excellent for detecting disk or network corruption, but it is the wrong tool for verifying software from an untrusted mirror unless the expected digest is trusted separately.


6. Where Checksums Are Used
Checksums and hashes show up at nearly every layer of a system. The diagram maps the main places they appear, and the sections below explain what they do in each place.




Checksums
and Hashes

Networking
Ethernet FCS
TCP / UDP

Storage
Disk blocks
WAL, backups

Databases
Pages
Replication logs

Object Storage
Upload integrity
Multipart parts

Packages
SHA-256 digests
Container images

Distributed Systems
Merkle trees
Replica repair

Networking
Ethernet frames include a frame check sequence, commonly a CRC. If a frame is damaged on the wire, the receiver can discard it.

IPv4 has a checksum for its header. IPv6 removed that IP-layer checksum and relies on lower and higher layers instead.

TCP and UDP also have checksums. They cover the packet header, payload, and some IP information used by the protocol. These checks catch accidental damage, not attacks. Stronger protection comes from TLS, QUIC, IPsec, or application-level signatures.

Storage Systems
Storage systems use checksums to detect corruption while data is stored and while it is being moved:

Disk blocks
Filesystem blocks
Database pages
Write-ahead logs
Object storage parts
Backup chunks
Recovery fragments that can rebuild missing data
Modern storage systems often keep checksums in metadata instead of trusting the data block alone. If a disk returns the wrong block without reporting an I/O error, the stored checksum can still catch the mismatch.

Databases
Databases use checksums on pages and logs to detect torn writes, disk corruption, and bad replication data.

If a database reads a page and the checksum fails, it should not pretend the page is valid. Depending on the system, it may read from a replica, restore from backup, stop the process, or mark the page corrupt.

Object Storage and Uploads
Object stores often let clients send a checksum with an upload. The service calculates its own checksum and rejects the object if the values differ.

For multipart uploads, be careful. Provider-specific ETags are not always a simple MD5 of the full object. Use the provider's documented checksum fields for integrity checks, not fields that merely look like checksums.

Package and Artifact Distribution
Package managers, container registries, and release systems use cryptographic hashes to identify released files.

Example:








1
The digest only helps if the expected value comes from somewhere you trust: a signed release manifest, package index, transparency log, or secure website.

Distributed Systems
Checksums and hashes help nodes compare data without sending every byte first.

Examples:

Replication systems compare checksums for log segments.
Replica repair can use hashes or Merkle trees to find ranges that differ.
Content-addressable storage names data by digest.
Deduplication systems avoid storing identical chunks twice.
Backup systems verify that restored data matches what was written.
At this layer, checksums are not just low-level plumbing. They become a design tool for finding differences cheaply.


7. End-to-End Integrity
A common mistake is assuming that a checksum at one layer protects the whole system. It does not.

A network frame checksum can show that one frame arrived intact on one link. It does not prove that the application wrote the right object, that a proxy returned the right file, or that storage kept the block intact months later.

For important data, use end-to-end integrity:

Compute a checksum or digest at the producer.
Store it with trusted metadata.
Verify it after every important boundary: upload, replication, storage, restore, and download.
Repair from another copy when verification fails.



Producer
compute digest

Upload
verify

Storage
verify on write/read

Replication
verify chunks

Consumer
verify final bytes

Distributed storage systems care about this because corruption can happen in many places: client memory, network cards, kernel buffers, disks, SSD firmware, replication jobs, compaction, backup pipelines, or restore tooling.


8. Checksums vs Encryption
Checksums and encryption solve different problems.

Checksums detect changes.

Encryption hides content from parties that do not have the key.

Modern secure protocols often provide both encryption and integrity checks. Still, the ideas are different. Encryption hides data. A checksum checks whether data changed. A checksum does not hide anything.

In practice:

Use CRCs for fast accidental corruption detection.
Use SHA-256 or another modern hash for content identity.
Use HMACs or signatures when you need to prove who produced the data.
Use TLS or another secure protocol for protected communication.
Use encryption at rest when stored data must stay private.

9. Design Considerations
Adding checksums to a system raises a few practical questions: how much data should one checksum cover, where should the checksum live, when should the system verify it, and what should happen when it does not match?

These choices decide whether checksums truly protect the data or merely add overhead.

Choose the Right Granularity
Checksumming an entire 10 GB object can tell you that something is wrong, but it will not tell you which part is bad. Checksumming 4 MB chunks lets a system retry or repair only the damaged chunk.

Smaller chunks make repair more precise, but they also create more metadata and more work during verification.

Store Checksums Where They Catch the Failure
If data and checksum are damaged together in the same way, verification may not help. Storage engines often keep checksums in page headers, metadata blocks, manifests, or separate indexes depending on the failures they want to catch.

The design question is: what failures are you trying to catch?

Verify on Both Write and Read
Verifying only during writes catches upload or transmission errors. Verifying on reads catches bit rot and silent corruption that happened while data was stored.

Cold data needs periodic checking because it may not be read for months. A backup that is never restored or verified is only a theory.

Do Not Ignore Mismatches
A checksum mismatch is not a warning to log and ignore. It means the bytes are not the bytes this layer expected.

Possible responses:

Retry the transfer.
Read another replica.
Reconstruct the data from recovery fragments.
Quarantine the object.
Mark the backup invalid.
Alert an operator.
Continuing with corrupt data often turns a contained data-integrity failure into a larger incident.


Summary
Checksums are compact values used to detect whether data changed. A match means no change was detected. It does not prove that change was impossible.

The kind of checksum matters. CRCs are strong tools for accidental corruption but not deliberate tampering. Cryptographic hashes such as SHA-256 are better for content identity and file verification. HMACs and digital signatures add proof that the data came from someone trusted.

Detection and recovery are separate jobs. Checksums tell you corruption happened. Replicas, retransmission, repair fragments, and backups are what recover the data.

To be useful, verification has to happen at system boundaries and during reads, not only when writing. A mismatch should be treated as a data-integrity failure, not harmless noise.

In long-running systems, some corruption is eventually expected. Checksums help catch bad bytes before they spread into application state, replicas, or backups.