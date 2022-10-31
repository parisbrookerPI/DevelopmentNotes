**Lecture 425**
#Listeners
#TDD

---
# Overview

## Theroy Points
- Listener setup
- Concurrency concerns
- Importance of reporting any changes to an object with events to all concerned service, particularly with the versioning strategy
- Locking services
- TDD

## Application-Specific

Setting up and testing TicketService listeners. Thsi time, we need to implement a ticket locking/unlocking function.

Essentially needs to react to `order:created` and `order:cancelled` events emitted from OrderCreatedPublisher and OrderUpdatedPublisher

In terms of implementing the lock, a simple lock flag won't provide enough information. The orderId would be a better flag for facilitating followup requests.

![[Pasted image 20221030154127.png]]

In code this means
- In `OrderCreatedListener.onMessage()`, take orderId and ticketId
- Fetch the ticket using ticketId
- On the Ticket document, store the orderId


---
# Update TicketService/ticketSchema


```TypeScript
//Ticket.ts
interface TicketDoc extends mongoose.Document {
  title: string;
  price: number;
  userId: string;
  version: number; //Updated __v to version
  orderId?: string;
}
...
const ticketSchema = new mongoose.Schema(
  {
    title: {
      type: String,
      required: true,
    },
    price: {
      type: Number,
      required: true,
    },
    userId: {
      type: String,
      required: true,
    },
    orderId: { type: String },
  },
  {
    toJSON: {
      transform(doc, ret) {
        ret.id = ret._id;
        delete ret._id;
      },
    },
  }
);
```

# Implement TicketService/OrderCreatedListener
```TypeScript
//OrderCreatedListener.ts
export class OrderCreatedListener extends Listener<OrderCreatedEvent> {
  readonly subject = Subjects.OrderCreated;
  queueGroupName: string = queueGroupName;
  async onMessage(data: OrderCreatedEvent["data"], msg: Message) {
    //Find ticket that the order is reserving
    const ticket = await Ticket.findById(data.ticket.id);
    // If no ticket, throw an error
    if (!ticket) {
      throw new Error("Ticket not found");
    }
    // Mark ticket as reserved by setting orderId property
    ticket.set({ orderId: data.id });
    // Save the ticket
    await ticket.save();
    // ack the message
    msg.ack();
  }
}
```

---
# Testing

```TypeScript
//OrderCreatedListener.test.ts
const setup = async () => {
  //create an instance of the listener
  const listener = new OrderCreatedListener(natsWrapper.client);
  // create and save a ticket
  const ticket = Ticket.build({
    title: "Concet",
    price: 94,
    userId: new mongoose.Types.ObjectId().toHexString(),
  });
  await ticket.save();
  console.log(ticket);
  //Create fake data event
  const data: OrderCreatedEvent["data"] = {
    id: new mongoose.Types.ObjectId().toHexString(),
    version: 0,
    status: OrderStatus.Created,
    userId: new mongoose.Types.ObjectId().toHexString(),
    expiresAt: "string;",
    ticket: {
      id: ticket.id,
      price: ticket.price,
    },
  };
  //create a fake message object
  // @ts-ignore
  const msg: Message = {
    ack: jest.fn(),
  };
  return { listener, data, msg, ticket };
};
it("sets the userId of the ticket", async () => {
  const { listener, data, msg, ticket } = await setup();
  await listener.onMessage(data, msg);
  const updatedTicket = await Ticket.findById(ticket.id);
  expect(updatedTicket!.orderId).toEqual(data.id);
});
it("acks the message", async () => {
  const { listener, data, msg } = await setup();
  //call the on message function with the data object + message object
  await listener.onMessage(data, msg);
  //write assertions to make sure ack is called
  expect(msg.ack).toHaveBeenCalled();
});
```

---
# Another consideration

Consider:
- This would be reserving the ticket.
![[Pasted image 20221030175918.png]]

Then if this order were cancelled:
![[Pasted image 20221030180103.png]]

Then the owner looks to edit the ticket:
![[Pasted image 20221030180150.png]]

And would emit this event, and sed it ot the order service
![[Pasted image 20221030180221.png]]

But, the order service is still on version 0
![[Pasted image 20221030180324.png]]

Any change to the ticket (i.e. thorugh an order) must be percolated to all other relevant services with an event.

## *Whenerver any change to a Record, we have to emit an event*

---

# Implement Messages for Consistent Versions Accross Services

- Update publisher interfaces for versioning.

## Another consideration around passing clients into publishers 

Current configuration (passing singleton stan) from natsWrapper:
![[Pasted image 20221030195018.png]]

There is in fact a stan client in the base class as a private property. If we change this to protected, then it could be accessed by child classes, especially handy for testing.
![[Pasted image 20221030195223.png]]

In common, Need to:
- `TicketUpdatedEvent` must have orderId property
- Clinet in base listener shouldbe protected, no private

---
# Testing
#jest - note how to look deeper inside the jest mock function call

```TypeScript
//filename.ts
const setup = async () => {
  //create an instance of the listener
  const listener = new OrderCreatedListener(natsWrapper.client);
  // create and save a ticket
  const ticket = Ticket.build({
    title: "Concet",
    price: 94,
    userId: new mongoose.Types.ObjectId().toHexString(),
  });
  await ticket.save();
  console.log(ticket);
  //Create fake data event
  const data: OrderCreatedEvent["data"] = {
    id: new mongoose.Types.ObjectId().toHexString(),
    version: 0,
    status: OrderStatus.Created,
    userId: new mongoose.Types.ObjectId().toHexString(),
    expiresAt: "string;",
    ticket: {
      id: ticket.id,
      price: ticket.price,
    },
  };
  //create a fake message object
  // @ts-ignore
  const msg: Message = {
    ack: jest.fn(),
  };
  return { listener, data, msg, ticket };
};
it("sets the userId of the ticket", async () => {
  const { listener, data, msg, ticket } = await setup();
  await listener.onMessage(data, msg);
  const updatedTicket = await Ticket.findById(ticket.id);
  expect(updatedTicket!.orderId).toEqual(data.id);
});
it("acks the message", async () => {
  const { listener, data, msg } = await setup();
  //call the on message function with the data object + message object
  await listener.onMessage(data, msg);
  //write assertions to make sure ack is called
  expect(msg.ack).toHaveBeenCalled();
});
it("publishes a ticket updated event", async () => {
  const { listener, data, msg } = await setup();
  await listener.onMessage(data, msg);
  expect(natsWrapper.client.publish).toHaveBeenCalled();
  // How to do deeper inspection of mockfunctions
  let ticketUpdatedData = (natsWrapper.client.publish as jest.Mock).mock
    .calls[0][1];
  ticketUpdatedData = JSON.parse(ticketUpdatedData);
  expect(data.id).toEqual(ticketUpdatedData.orderId);
});
```

---
# Implement OrderUpdated Listener

- Prettry much a clone of the OrderCreated listener

```TypeScript
import { Listener, OrderCancelledEvent, Subjects } from "@parisbtickets/common";
import { queueGroupName } from "./QueueGroupName";
import { Message } from "node-nats-streaming";
import { Ticket } from "../../models/Ticket";
import { TicketUpdatedPublisher } from "../publishers/TicketUpdatedPublisher";

export class OrderCreatedListener extends Listener<OrderCancelledEvent> {
  readonly subject = Subjects.OrderCancelled;
  queueGroupName = queueGroupName;
  async onMessage(data: OrderCancelledEvent["data"], msg: Message) {
    const ticket = await Ticket.findById(data.ticket.id);
    if (!ticket) {
      throw new Error("Ticket nopt found");
    }
    ticket.set({ orderId: undefined });
    await ticket.save();
    await new TicketUpdatedPublisher(this.client).publish({
      id: ticket.id,
      orderId: ticket.orderId,
      version: ticket.version,
      title: ticket.title,
      price: ticket.price,
      userId: ticket.userId,
    });
  }
}
```

---
# Locking Down a Record in RouteHandler

- Prettry much just add a check for orderId

---
