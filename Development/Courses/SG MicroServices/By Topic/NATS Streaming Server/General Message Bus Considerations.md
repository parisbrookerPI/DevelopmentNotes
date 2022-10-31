### Multiple Scale-out Subscribers to Same Topic
- If two copies of same service are subscrbing to one topic, it may not be desireabel for them to poerform the same processsign on the same copy of a message. way around this is to use queue groups (in NATS - what is equivalent for Service Bus?)

### Ephemeral Nature of Messages
- Default behaviour is to lose the event, they are not persisted (how to do this in Service bus?).

### Handling Client Shutdown

### **Maintaining Core Concurrency (Lecture 309/310/311)**
- There are multiple ways a service might handle an event/message out of order, and this can be catastrophic.
- **This is actually the fundamental difficulty of data handling within microservice applications**

Solutions?
1) (Not viable):  Run copy of the Service
2) (Not viable):  Handle every case in code
3) (Not viable): Shared state between cloned services - can only process one update at a time
4) (not viable): Track events by an id. This would require channels per ID. Not scaleable
5) (not viable): Move ID and sequence (i.e. last-modified field) into persistent storage for a service (with NATS sending 'last ID' info back to service)

The (Best, not Perfect) Solution:
- The fundamental design of the system needs to be rethought

![[Pasted image 20221016124859.png]]

![[Pasted image 20221016125108.png]]

The accounting Example exhibits poor design:
![[Pasted image 20221016125248.png]]
- What is the publisher responsible for?
- Where are the events coming from?
- What is the data? 

A better approach would be:
![[Pasted image 20221016125405.png]]

1) ![[Pasted image 20221016125533.png]]
2) ![[Pasted image 20221016125846.png]]
3) ![[Pasted image 20221016125958.png]]
4) ![[Pasted image 20221016130157.png]]
5)  If the last Trx test fails, the out-of-order event is disregarded, then the in order is retried
----
## Implementing the Concurrency Solution

![[Pasted image 20221016131805.png]]

---
## Testing 