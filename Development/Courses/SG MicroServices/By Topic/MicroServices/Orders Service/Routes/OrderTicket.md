How do we relate an Order to a Ticket?

![[Pasted image 20221022125032.png]]

![[Pasted image 20221022125052.png]]

Downsides:
- When a user makes a request to create an order, need to ensure thatticket isn't reserved... tricky to do with a query 
![[Pasted image 20221022125413.png]]

- Can't deal with events, because would require another pool of unpurchased tickets
![[Pasted image 20221022125537.png]]

Second option, separate document collections, using  ref/population:
![[Pasted image 20221022125619.png]]



![[Pasted image 20221022134018.png]]
