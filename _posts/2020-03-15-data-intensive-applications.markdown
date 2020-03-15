---
layout: post
title:  "Designing data intensive applications"
date:   2020-03-15 18:30:48 +0530
categories: notes
---

* TODO Designing data intensive applications
    :NOTER_DOCUMENT: /Users/deepak/ebooks/reading list/Martin Kleppmann-Designing Data-Intensive Applications_ The Big Ideas Behind Reliable, Scalable, and Maintainable Systems-O’Reilly Media (2017).

** Chapter 1 - Reliable, Scalable and Maintainable Applications
*** Reliability

**** The system should continue to work correctly (performing the correct function at the desired performance) even in the face of adversity (hardware or software faults, and even human error). 
**** Only makes sense to talk about certain types of faults. Eample of a tool for testing fault tolerance - Netflix's chaos monkey .
**** Hardware faults - MTTF of hard disks if 10 to 50 years so add *redundant* hardware (disks, servers)
**** Software faults - Bugs, runaway process, cascading failures.
**** Human errors - To reduce these, reduce complexity, improve design, setup monitoring.

*** Scalability
    As the system grows (in data volume, traffic volume or complexity), there should be reasonable ways of dealing with that growth.
**** Describing load - Twitter example
***** Post tweet - 4.6 rps on avg, 12k rps at peak.
***** Home timeline - 300k rps
***** Approach 1 to fetch timeline - Get all the followees, get their tweets sroted by time and return the result. It struggled to keep up with the load of the queries.
***** Approach 2 - Maintain a cache of timelines. Whenever a tweet is created, push it in the follower's cache. Problem of *fanout* in case of celebrities.

A hybrid of 1 and 2 is used in practice.

**** Describing performance
***** When you increase a load parameter, and keep the system resources (CPU, memory, network bandwidth, etc.) unchanged, how is performance of your system affected?
***** When you increase a load parameter, how much do you need to increase the resources if you want to keep performance unchanged?
***** In batch processing systems like Hadoop we care about throughput, in online systems we care about response times.

*Latency and response time are often used synonymously, but they are not the same. The response time is what the client sees: besides the actual time to process the request (the service time), it includes network delays and queueing delays. Latency is the duration that a request is waiting to be handled — during which it is latent, awaiting service*

***** Better to user percentiles instead of averages.
***** Cope with load - Distributing load across multiple machines is also known as a shared nothing architecture.

*** Maintainability
    Over time, many different people will work on the system (engineering and oper‐ ations, both maintaining current behavior and adapting the system to new use cases), and they should all be able to work on it productively.
**** Operability - Make it easy for operations teams to keep the system running smoothly.
**** Simplicity - Make it easy for new engineers to understand the system, by removing as much complexity as possible from the system. (Note this is not the same as simplicity of the user interface.)
**** Evolvability - Make it easy for engineers in future to make changes to the system, adapting it for unanticipated use cases as requirements change. Also known as extensibility, modifiability or plasticity.

** Chapter 2 - Data Models and query languages
*** The best-known data model today is probably that of SQL, based on the relational model data is organized into relations, where each relation is an unordered collection of tuples (rows). The goal of the relational model was to hide that implementation detail behind a cleaner interface.
*** NoSQL
*** Object-relation mismatch
**** SQL model - table, columns,
**** Later versions of the SQL standard added support for structured datatypes and XML data, which allow multi-valued data to be stored within a single row
**** Encode jobs, education and contact info as a JSON or XML document, store it on a text column in the database, and to let the application interpret its structure and content
*** Relationships - have to manage joins in application code in nosql
*** Types of models
**** Network model -  In the tree structure of the hierarchical model, every record has exactly one parent; in the network model, a record can have multiple parents. The links between records in the network model are not foreign keys, but more like pointers in a programming language
**** Relational model -  In a relational database, the query optimizer automatically decides which parts of the query to execute in which order, and which indexes to use. Those choices are effec‐ tively the “access path”, but the big difference is that they are made automatically by the query optimizer, not by the application developer, so we rarely need to think about them.
**** Document model - The main arguments in favor of the document data model are: for some applications it is closer to the data structures used by the application, schema flexibility, and better performance due to locality. The relational model counters by providing better sup‐ port for joins, many-to-one and many-to-many relationships
***** The schema-on-read approach is advantageous if the data is heterogeneous
*****  If your application often needs to access the entire document (for example, to render it on a web page), there is a performance advantage to this storage locality.

*** Query languages
**** An imperative language tells the computer to perform certain operations in a certain order. You can imagine stepping through the code, line by line, evaluating condi‐ tions, updating variables, and deciding whether to go around the loop one more time.
**** In a declarative query language, like SQL or relational algebra, you just specify the pattern of the data you want — what conditions the results must meet, and how you want it to be transformed (e.g. sorted, grouped and aggregated), but not how to ach‐ ieve that goal. It is up to the database system’s query optimizer to decide which indexes and which join methods to use, and in which order to execute various parts of the query.
**** Declarative languages have a better chance of getting faster in parallel execution, because they specify only the pattern of the results, but not the algorithm that is used to determine the results. The database is free to use a parallel implementation of the query language, if appropriate 

*** Graph like data models
Facebook maintains a single graph with many different types of vertex and edge: vertices represent people, locations, events, checkins and comments made by users; edges indicate which people are friends with each other, which checkin hap‐ pened in which location, who commented on which post, who attended which event, etc.

**** Property graphs
***** vertex - unique id, outgoing edges, incoming edges, collection of properties
***** edge - unique id, start and ende vertices, label to describe relationship, collection of properties
**** query language - cypher (declarative)
**** Graph queries in sql also possible - In Cypher, :WITHIN*0.. expresses that fact very concisely: it means “follow a WITHIN edge, zero or more times”. SQL has to use keyword "recursive".

*** Triple stores
**** (subject, predicate, object). - (Jim, likes, bananas). Object can be another vertext or can also be a value to a property on the subject node.
**** The Resource Description Framework (RDF) was intended as a mechanism for different web‐ sites to publish data in a consistent format, allowing data from different websites to be automatically combined into a web of data, a kind of internet-wide ‘database of everything’.




** DONE Chapter 5 - Replication
   CLOSED: [2018-04-15 Sun 18:40] DEADLINE: <2018-04-14 Sat>
*** Multi leader configuration - say you have two DCs and each has its own leader which can accept read/write queries. Each leader replicates the data to the leader in other DC

    Advantages:
    Better performace as there are multi DCs to accept writes
    DC outage can be tolerated
    Network issues between DCs can be tolerated. Other DC can catch up asynchronously if there are network issues between DCs

    Disadvantages:
    Writes can happen concurrently to different DCs

    Handling conflict:
    Conflict avoidance - Make sure that all the writes from one user go to the same DC. Each user (a record / piece of data being edited) having a "home" DC. This way conflict can't happen.

    Ways to achieve a convergent state:
    - Give each write an unique ID (timestamp, UUID, random number) and the write with the highest ID wins - can lead to data loss
    - Give each write an unique ID and write from a higher numbered replica gets precendence - also leads to data loss
    - Somehow merge the two writes - may be concatenate them in alphabetical order
    - Record the conflict in a special data structure and write conflict resolution in application (mabye prompting the user)


**** TODO Read about [[https://en.wikipedia.org/wiki/Operational_transformation][Operation Transformation]] - Google docs algorithm for conflict resolution

     There can be multiple topologies for multi leader configurations - circular, start, all-to-all. Circular and start have single point of failure. All-to-all does not have single point
     of failure but different links may be diffferent speeds dues to network congestion and hence writes may arrive out of order at different leaders. E.g., an insert is done at leader 1
     and it forwards it to leaders 2 and 3. 2 processed it but it is still on the way to 3. Now update occurs on same record and this write reached 3 but the insert is still on the way.
     So at 3 there will be a case of update on a record that does not even exist!

*** Leaderless repllication
    The client sends read/write request to multi replicas. If any replica is down during a write it misses the write. A version number is used to determined which is the updated value when
    a read occurs (becase now the replica which missed the write will return a value with an older version number)

    How do replicas that missed writes catch up?
    - Read repair - While reading client detects that a replica has stale value and writes back the new value to it - works well for frequently read values
    - Anti entropy - A background process that looks out for missing data and copies

    Quorum reads and writes
    Writes should be confirmed by atleast w nodes and reads would be confirmed by r nodes - as long as w + r > n we are good.
    When writes are few and reads frequent use w=n and r=1 but this has the disadvantage that any one node failure will cause writes to fail


    Limilations of quorum consistency
    - On concurrent read and write it is not clear that which value the read should return
    - When less than w returned ok, the writes in those other nodes which succeeded are not rolled back
    - When a node with a new write fails and its data is restored from a replica having old value the quorum condition is broken

    Algorithm for not losing concurrent writes:
    - Server maintains a version number for each key, increments the version numbe each time a key is written and stores new version number along with the new value
    - On a read the server returns all values which are not overwritten along with the latest version number
    - When a client writes it must include the version number obtained in the previous read and also must merge laa the values received in prior read
    - When a server receives a write for a key it overwrites all the version less than or equal to the received version number, but must retain all the greater version numbers
      as they are concurrent writes

    In case of multi replica configurations "Version vectors" are used which are just versions per per replica. A replica receiving a write also receives versions from other replicas
    on that key and figures out which values to overwrite and which ones to keep as siblings

** DONE Chapter 6 - Partitioning
   CLOSED: [2018-04-17 Tue 00:02]

   How to decide which records to store on which node? Simplest approach is to randomly distribute the data across nodes but has the disadvantage that while reading we don't know where
   the data is so we have to query all the nodes parallely.

   *Range based partitioning* - e.g., keys with values A-B are stored on one partition, C-F on another and so on. Withing each partition the keys can be sorted for easy scanning.
   The downside is that it may create /hot spots/ as the data may not be evenly distributed

   *Partitioning by hash of key* - Use a 32 bit hash function and distribute keys on nodes according to their hash values; this ensures distributed data on nodes as a good hash function
   distributes data seemingly randomly even if the input is similar. The downside is that range queries are now have to be sent on all partitions. In range based partitions range based
   queries could be answered by adjacent keys but now thay are scattered all over

   Hashing based on compound primary key (multi column primary key) - Hashing is done based on the first part of the key so that range queries based on the remaining key are efficient.
   E.g., a user may be many updates so store data as (user_id, timestamp) so all the data from same user_id goes to same partition and in that partition the tweets are ordered by
   timestamp.

   Secondary indexing - Efficiently search a documents based on something other than its primary key
   Local/Document indexing - the docuemnts in one partition are secondary indexed (based on which 'columns' will be queried) in the same partition. Now queries like give all docs with
   color red will have to be send read on all partitions
   Global/term indexing - there is a separate index which is again partitioned by the term or hash of term. For querying just go to that partition (by getting hash of that term) to read.
   But the downside is that any update to a doc will possibly have to write to multiple partitions

   
   *** Rebalancing
   
   Use range based hashing i.e., if hash(key) is between b0 and b1 put it in bucket 0, if between b1 and b2 put it in bucket 1 and so on. Do not use mod n because when n changes
   a large number of keys have to be displaced
   Fixed number of partitions - 10 nodes, 1000 partitions - roughly 100 partitions per node. Fix the number of paritions. Even if nodes increase, the new node will /steal/ partitions
   from other nodes and when nodes decrese the process will happen in reverse
   Dynamic partitioning - Partitions are split and merged as data grows and decreases. A split partition can even move to another node.



** DONE Chapter 7 - Transactions
   CLOSED: [2018-04-23 Mon 00:53] DEADLINE: <2018-04-22 Sun>
   
   *** Transaction isolation levels
   
   **** Read committed isolation level
   - No dirty reads - While a write is going on on an object (row or document) and it has not yet committed, a read on that object should not read uncommited value. It is implemented by the
     database by remembering both the old and new values. While a write is going on, and a read comes the db will return the the old value.
   - No dirty writes - While a write is going on and has not committed, another write is not allowed. It is implemented by row level locks. A transaction aquires lock on the rows/documents
     it wants modify and releases after it has committed

     Problem - Repeatable read or read skew.
     If Alice has 1000 balance split 500 each at two different accounts. Now she transfers 100 from on to another. If, during this transaction, she reads the two values such that from debting
     account 100 have been debited and have not been received at the other end, she will see 400 and 500 respectively in each. This is acceptable in read committed isolation because when she
     read the values they were indeed committed.

   **** Snapshot isolation
   Read/writes happen on a consistent snapshot of the database. DB has to maintain multiple versions of an object because multiple transactions may be reading/writing from/to different
   version of the database. This is useful in applications like backups where the data is being modified as it is being backed up. To the backup process must see a consistent snapshot
   of data otherwise if the data is being modified simultaneously then the backup may become nonsensical.

   The db maintains multiple versions of an object. Each row has a created by id (id of the transaction whic created the row), and a deleted by id (which is null initially and is assigned
   the id of the transaction which requested its deletion). Updates are internally converted to delete followed by a write. A garbage collector process clears the deleted row when certain
   that no transaction can access this data now.
   ***** Rules for when an object is visible to a transaction
   - If the object that is being read is created by a transaction that has finished (committed) before the transaction to read started, i.e., the id in the created by field of the object being read is
     less than the current transaction's id.
   - The object is not marked for deletion or if it is the deleting transaction has not committed yet.

   *** Problem of lost updates - e.g., incrementing a counter. Two transactions read a value, say 42, get intertwined and both write 43 which should have been 44
   **** Atomic write operations - DB itself provides atomic updates so that application code does not have to handle unsafe read-modify-write cycles. Implemented by ontaining lock on the object.
   **** Explicit locking - Application code locks all the objects necessary. Easy to forget obtaining a lock at arequired place.
   **** Auto detecting lost updated - Transaction manager automatically detects a lost update and aborts the transaction if an update is lost
   - Compare and set - It allows an update to proceed if the value that is read has not changed at the time update is begin done.

  **** Generalized case of lost updates - In lost updates, an update to an object is lost when two transactions try to modofy the same object. In the general case two transactions are unaware
  of what the other is doing with multiple rows instead of just one object. Following is the pattern:
  - A search query to check is a requirement is met (like the meeting room is not booked for a certain period of time)
  - Based on the previous step decode what to do
  - Insert rows
  - Now if the first select query were to run again, it would give different results.
  
  Like lost updates, on transanction has changed the result of the select query (in lost update a transaction changes the state of an object). The other transaction is unaware of this, and if these
  two trnasactions intertwine then multiple meeting bookings can be made for the same meeting room.

  ***** Materializing conflicts - If there is no pbject to lock, create a lock in the database. E.g., for meeting room case create a table in which rows correspons to every possible combination of
  rooms and time slots. This table does not contain any data it's just there for locking purpose. Whenever a meeting room is booked, aquire a lock on that certain row in the lock table. Of course
  this method is ugly and should be used as a last resort.

  **** Serializability - Strongest isolation guarentee. Behaves as if two transactions have executed serailly
  ***** Actual serial execution
  ***** 2PL - two phase locking
  - Several transactions can simultaneously read as long as no one is writing. But if any one writing then locks are required. If an object is being written then readers of that object are blocked,
    and also if object is being read then writers to that object are blocked. This is different than snapshot isolation where writers dont block readers and readers dont block writers. In 2pl, 
    writers blocks aother writers (which is same in snapshot) and readers block writers and also writers block readers. 2PL gets its name from - first phase aquire all locks, second phase release all
    locks. Two types of locks - shared mode and exclusive mode. Shared mode for reading object, exlucsive mode for writing object. Every object has a lock.

   - Predicate locking - solution for meeting room booking
     Locks are not obtained on a single object but on all objects which match a predicate, e.g, the result of select query for a particular room at a particular time. The key idea is that is can hold
     locks on objects which do not exist yet i.e., phantoms. When transaction wants to insert (new value), modify, delete it must check whether any transaction holds a lock on those objects. E.g., 
     While inserting the object is not even there but still it must check if any other transaction could hold lock on that object as that object migh match the predicate which the other transaction
     is holding a lock on
   - Index-range locks - Approximation of predicate locking. Locks a superset of what predicate locking locks. In meeting booking case, if you want room 123 from noon to 1, predicate locking would
     locks room 123 and time range noon to 1, while index-range locking would lock room 123 at all times or lock all rooms at noon to 1. Idea is that room would already have an index so attach
     a shared lock to it, another transaction trying to modify would see that a shared lock has been obtained and would wait for it to be released. If no idexes are there then locking the whole
     table is an options but that is very costly in terms of performance.

  ***** Serializable snapshot isolation - Optimistic appraoch - Let a trnsaction through and the db checks if isolation may have been violated and aborts the transactoin which needs to be retried
  by the application
    
** DONE Chapter 7 - The trouble with distributed systems
   CLOSED: [2018-04-29 Sun 20:19] DEADLINE: <2018-04-28 Sat>

*** Clocks 
    Time-of-day clock - System.currentTimeMillis() in java; measure time from 1jan 1970 utc midnight and does not count leap seconds. Can jump back in time due to a sunchronization using NTP 
    (network time protocol). Gives "absolute value of time" (more or less), 
    and Monotonic clock - System.nanoTime() in java; measure time from some arbitrary point in time (maybe when the syatem was restarted); Used to measure a duration. It always increases unlike
    time-of-day which can jump back in time.

    In a single leader configuration how does the leader know that it has not been declared dead by other nodes?
    Obtain a lease from other nodes which is like a lock with a timeout. Only one node can hold a lease at a time. But what if a client which holds the lease undergoes a large GC pause and in that
    time the lease expires? When the client recovers from the GC pause it still wrongly believes that it is the one holding the lease but in the mean time some other client has been given the lease.
    Now both of clients will try to write and will corrupt the file.
    Solution - A fencing token (monotonically increasing) is also sent along with the lease and the client has to present the token every time doing an operation

** TODO Chapter 8 - Consistency and Consensus
   SCHEDULED: <2018-05-01 Tue>
** TODO Chapter 10 - Batch processing
** TODO Chapter 11 - Stream Processing

