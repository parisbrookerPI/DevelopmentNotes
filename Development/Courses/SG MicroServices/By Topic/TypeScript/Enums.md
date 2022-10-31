A way to define constants in a location, and have these refered to in other locations while being checked by TypeScript . Working with interaces, they can create a tight loop of Type checking.

### Useage Examples:

Type checking event types in eventlistener subclass

![[Pasted image 20221016155028.png]]

```TypeScript 
//Subjects.ts
export enum Subjects {
    TicketCreated = 'ticket:ceated',
    OrderUpdated = 'order:updated'
}
```

```TypeScript
//TicketCreatedEvent.ts
import { Subjects } from "./Subjects";
export interface TicketCreatedEvent {
  subject: Subjects.TicketCreated;
  data: {
    id: string;
    title: string;
    price: number;
  };
}

```

Incorporating these into the even listener allows us to check we're **listening to the right event** and **handling the right type of data**, but this will require an other interface defining the shape of an Event, and designating Listener (the parent class) as a generic class.

```TypeScript
//BaseListener.TS
interface Event {
  subject: Subjects;
  data: any;
}
export abstract class Listener<T extends Event> { // Interface passed into calls like argument 
   abstract subject: T["subject"]; //must be of type refered to by the key 'subjects' in interface
  abstract queueGroupName: string;
  abstract onMessage(data: T["data"], msg: Message): void; //must be of typp refered to by key 'data' in interface
  private client: Stan;
  protected ackWait = 5 * 1000;
  ...
  }
```

Defining a class as generic `<T extends Event>`:
- Means we have to supply a custom type to the classs. They're like arguments for Types. So when the base class gets called by a subclass, we need to pass in the interface as an  argument

```TypeScript
//TickeCreatedListener.TS

import { Listener } from "./BaseListener";
import nats, { Message, Stan } from "node-nats-streaming";
import { TicketCreatedEvent } from "./TicketCreatedEvent";
import { Subjects } from "./Subjects";

export class TicketCreatedListener extends Listener<TicketCreatedEvent> {//TicketCreatedEvent shares interface properties with Event, so is valid 'argument' to the generic class declaration
  readonly subject = Subjects.TicketCreated; //readonly assures TypeScript this won't be rediefined later... like: 
  queueGroupName = "payments-service";
  onMessage(data: TicketCreatedEvent["data"], msg: Message): void {//will not check types for the TicketCreatedEvent interface 
    console.log("Business Logic Here on the", data);
    msg.ack();
  }
}
```
