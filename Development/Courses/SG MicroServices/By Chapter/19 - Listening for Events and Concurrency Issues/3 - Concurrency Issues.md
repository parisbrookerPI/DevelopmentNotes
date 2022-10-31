**See Lecture 401**

Solution to the concurrency issue is the versioning of the records between service dbs.This is facilitated by incrementing versions on messages. Mongoose and Mongo can do this natively.

# OCC in Mongo and Mongoose

![[Pasted image 20221030093246.png]]

![[Pasted image 20221030093526.png]]
- Need to turn on version incrementing in mongoose

![[Pasted image 20221030102745.png]]


----
# Implemention OCC in Mongoose

#Module: mongoose-update-if-current
#Setup: Mongoose plugin configuration
#MongooseConfig

- Import module
- Set as plugin
- Update `__v` field to `version`
- Update Document interface

Implemnetation in TicketService

```TypeScript
interface TicketDoc extends mongoose.Document {
  title: string;
  price: number;
  userId: string;
  version: number //Updated __v to version
}
// the build function that allows TypeScript to be happy
interface TicketModel extends mongoose.Model<TicketDoc> {
  build(attrs: TicketAttrs): TicketDoc;
}
const ticketSchema = new mongoose.Schema(
  ...
);
//Changes __v to version
ticketSchema.set("versionKey", "version");
// Initializes plugin
ticketSchema.plugin(updateIfCurrentPlugin);
```

---
# Testing
#Testing

```TypeScript
import { Ticket } from "../Ticket";
 
it("implements OCC", async () => {
  //Create and instance of a ticket
  const ticket = Ticket.build({
    title: "Concert",
    price: 5,
    userId: "23fsdf",
  });
  //Save the ticket to db
  await ticket.save();
  //Fetch ticket twice
  const firstInstance = await Ticket.findById(ticket.id);
  const secondInstance = await Ticket.findById(ticket.id);
  //make two separate changes to the tickets we fetched
  firstInstance!.set({ price: 10 });
  secondInstance!.set({ price: 15 });
  //save the first fetched ticket
  await firstInstance!.save();
  // save the second fetched ticket and expect an error
  try {
    await secondInstance!.save();
  } catch (err) {
    return;
  }
  throw new Error("Should not reach");
});

it("increments version by 1", async () => {
  const ticket = Ticket.build({
    title: "Concert",
    price: 5,
    userId: "23fsdf",
  });
  await ticket.save();
  expect(ticket.version).toEqual(0);
  await ticket.save();
  expect(ticket.version).toEqual(1);
  await ticket.save();
  expect(ticket.version).toEqual(2);
});

```

---
# Updating the Common Module

- This is necessary because we need to add the `version` field to all Event interfaces
- Need to hunt for event publish events and ensure `version` is included

---
# Updating listener logic
```TypeScript
export class TicketUpdatedListener extends Listener<TicketUpdatedEvent> {
  readonly subject = Subjects.TicketUpdated;
  queueGroupName = queueGroupName;
  async onMessage(data: TicketUpdatedEvent["data"], msg: Message) {
    //Find the ticket, update it, save it
    // const ticket = await Ticket.findById(data.id);
    //With addition of the versioning solution, we need to use a more granular query
    const ticket = await Ticket.findOne({
      _id: data.id,
      version: data.version - 1,
    });
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
# OrderService Listener Testing
```TypeScript
//TicketCreatedListener.test.ts
import { natsWrapper } from "../../../NatsWrapper";
import { TicketCreatedEvent } from "@parisbtickets/common";
import { TicketCreatedListener } from "../TicketCreatedListeners";
import { Message } from "node-nats-streaming";
import { Ticket } from "../../../models/Ticket";
import mongoose from "mongoose";

const setup = async () => {
  //create an instance of the listener
  const listener = new TicketCreatedListener(natsWrapper.client);
  // create a fake data event
  const data: TicketCreatedEvent["data"] = {
    version: 0,
    id: new mongoose.Types.ObjectId().toHexString(),
    title: "Concet",
    price: 94,
    userId: new mongoose.Types.ObjectId().toHexString(),
  };
  //create a fake message object
  // @ts-ignore
  const msg: Message = {
    ack: jest.fn(),
  };
  return { listener, data, msg };
};
it("creates and saves a ticket", async () => {
  const { listener, data, msg } = await setup();
  //call the on mmessage function with the data object + message object
  await listener.onMessage(data, msg);
  //write assertions to make sure a ticket was created
  const ticket = await Ticket.findById(data.id);
  expect(ticket).toBeDefined();
  expect(ticket!.title).toEqual(data.title);
  expect(ticket!.price).toEqual(data.price);
});
it("acks the message", async () => {
  const { listener, data, msg } = await setup();
  //call the on message function with the data object + message object
  await listener.onMessage(data, msg);
  //write assertions to make sure ack is called
  expect(msg.ack).toHaveBeenCalled();
});
```

```TypeScript
//TicketUpdeatedListener.test.ts
import { natsWrapper } from "../../../NatsWrapper";
import { TicketUpdatedListener } from "../TicketUpdatedListener";
import { TicketUpdatedEvent } from "@parisbtickets/common";
import { Message } from "node-nats-streaming";
import { Ticket } from "../../../models/Ticket";
import mongoose from "mongoose";

  

const setup = async () => {
  //create an instance of the listener
  const listener = new TicketUpdatedListener(natsWrapper.client);
  //Build a ticket
  const ticket = Ticket.build({
    id: new mongoose.Types.ObjectId().toHexString(),
    price: 333,
    title: "some show",
  });
  await ticket.save();
  // create a fake data event
  const data: TicketUpdatedEvent["data"] = {
    version: ticket.version + 1,
    id: ticket.id,
    title: ticket.title,
    price: 94,
    userId: new mongoose.Types.ObjectId().toHexString(),
  };
  //create a fake message object
  // @ts-ignore
  const msg: Message = {
    ack: jest.fn(),
  };
  return { listener, data, msg, ticket };
};
it("finds, updates and saves", async () => {
  const { listener, data, msg } = await setup();
  //call the on mmessage function with the data object + message object
  await listener.onMessage(data, msg);
  //write assertions to make sure a ticket was created
  const updatedTicket = await Ticket.findById(data.id);
  expect(updatedTicket!.title).toEqual(data.title);
  expect(updatedTicket!.price).toEqual(data.price);
  expect(updatedTicket!.version).toEqual(data.version);
});
it("acks the message", async () => {
  const { listener, data, msg, ticket } = await setup();
  //call the on message function with the data object + message object
  await listener.onMessage(data, msg);
  //write assertions to make sure ack is called
  expect(msg.ack).toHaveBeenCalled();
});

it("does not call ack if events are out of sync", async () => {
  const { listener, data, msg, ticket } = await setup();
  data.version = 10;
  try {
    await listener.onMessage(data, msg);
  } catch (err) {}
  //write assertions to make sure ack is called
  expect(msg.ack).not.toHaveBeenCalled();
});
```

---
![[Pasted image 20221030151308.png]]