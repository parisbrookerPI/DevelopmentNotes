![[Pasted image 20221029141721.png]]

Find Order Cancelled image! 


**Theory Extraction** 
- What are the events that can happen in a service?
- Think about what other services need to know about an event in a particular service
- Base publisher and listener classes should be defined in a common module
- Event interfaces should be defined in a common module, event subjects defined in an enum, and any statuses defined in an enum

**Procedure**
- Implement publisher after business logic
- Test that the publisher function is called
