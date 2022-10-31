![[Pasted image 20221030082004.png]]

The information in the event emitted by the tickets service is replicated in the Orders Service datastore to avoid synchronous comms.

# OrderService/TicketCreatedListener
## Implementation - Itteration 1:
``` TypeScript
export class TicketCreatedListener extends Listener<TicketCreatedEvent> {
  readonly subject = Subjects.TicketCreated;
  queueGroupName = queueGroupName;
  async onMessage(data: TicketCreatedEvent["data"], msg: Message) {
    const { title, price } = data;
    const ticket = Ticket.build({
      title,
      price,
    });
    await ticket.save();
    msg.ack();
  }
}
```

There's no consideration of the  `id / _id` property on the ticket ... this is required to replicate truly the records between the ticket dbs in TicketService and OrderService.

We have to account for the removal of the `_` previously implemented in the ticketSchema.
**NOTE: Mongoose will not consider `id` as a valid `_id` property; it will just add `_id`. This requires the `Attrs` interface and `build()` method on the model to be adjusted **

```TypeScript
// Initial implementation
export class TicketCreatedListener extends Listener<TicketCreatedEvent> {
  readonly subject = Subjects.TicketCreated;
  queueGroupName = queueGroupName;
  async onMessage(data: TicketCreatedEvent["data"], msg: Message) {
    // Look at data object for title and price
    const { title, price, id } = data;
    const ticket = Ticket.build({
      title,
      price,
      id,
    });
    await ticket.save();
    msg.ack();
  }
}
```

---
# OrderService/TicketUpdatedListener
## Implementation - Itteration 1

```TypeScript
export class TicketUpdatedListener extends Listener<TicketUpdatedEvent> {
  readonly subject = Subjects.TicketUpdated;
  queueGroupName = queueGroupName;
  async onMessage(data: TicketUpdatedEvent["data"], msg: Message) {
    //Find the ticket, update it, save it
    const ticket = await Ticket.findById(data.id);
    if (!ticket) {
      throw new Error("Ticket not found");
    }
    const { title, price } = data;
    ticket.set({ title, price });
    await ticket.save();
    msg.ack();
  }
}
```


---
# How to initialize the listeners

- Remember, the listener requires a stan to passed into it before it can start listening
- Can initialize the listeners inside index.ts
```Typescript
 await natsWrapper.connect(
      process.env.CLUSTER_ID,
      process.env.NATS_CLIENT_ID,
      process.env.NATS_URL
    );
    natsWrapper.client.on("close", () => {
      console.log("Nats conn closed");
      process.exit();
    });
    process.on("SIGINT", () => natsWrapper.client!.close());
    process.on("SIGTERM", () => natsWrapper.client!.close()); // Good practice not to hide this away in a class file
    //Set up event listeners:
    new TicketCreatedListener(natsWrapper.client).listen();
    new TicketUpdatedListener(natsWrapper.client).listen();
```

