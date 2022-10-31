Base Class reminder:

``` TypeScript
import nats, { Message, Stan } from "node-nats-streaming";
import { Subjects } from "./Subjects";

interface Event {
  subject: Subjects;
  data: any;
}

//Provide extended event type
export abstract class Listener<T extends Event> {
  abstract subject: T["subject"];
  abstract queueGroupName: string;
  abstract onMessage(data: T["data"], msg: Message): void; //Recieves the event
  private client: Stan;
  protected ackWait = 5 * 1000;
  constructor(client: Stan) {  //must provide the nats client in constructor
    this.client = client;
  }
  subscriptionOptions() {
    return this.client
      .subscriptionOptions()
      .setDeliverAllAvailable()
      .setManualAckMode(true)
      .setAckWait(this.ackWait)
      .setDurableName(this.queueGroupName);
  }
  //this method needs to be called to actually listen --subscribe--- for events
  listen() {
    const subscription = this.client.subscribe( 
      this.subject,
      this.queueGroupName,
      this.subscriptionOptions()
    );
    //when msg is received, this is called
    subscription.on("message", (msg: Message) => {
      console.log(
        `Message received: ${this.subject} / QueueGroup: ${this.queueGroupName}`
      );
      const parsedData = this.parseMessage(msg);
      this.onMessage(parsedData, msg);
    });
  }
  parseMessage(msg: Message) {
    const data = msg.getData();
    return typeof data === "string"
      ? JSON.parse(data)
      : JSON.parse(data.toString("utf8"));
  }
}

```

Listener Implementation:

```TypeScript

import { Message } from "node-nats-streaming";
import { Subjects, Listener, TicketCreatedEvent } from "@parisbtickets/common";
import { Ticket } from "../../models/Ticket";

export class TicketCreatedListener extends Listener<TicketCreatedEvent> {
  readonly subject = Subjects.TicketCreated;
  queueGroupName = "orders-service";
    onMessage(data: TicketCreatedEvent["data"], msg: Message) {
  }
}
```

Queue Group:
![[Pasted image 20221030081309.png]]
**Be careful with queue group name - perfect for a simple const export**

`onMessage(data: TicketCreatedEvent["data"], msg: Message) `
- `TicketCreatedInterface` 'data' property - Basically types the argument
- `Message` type contains the `ack()` method, so lets us call `ack()` 


