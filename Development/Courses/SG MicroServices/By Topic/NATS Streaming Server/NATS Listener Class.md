![[Pasted image 20221016152049.png]]
![[Pasted image 20221016151923.png]]

Base class:
```typescript
abstract class Listener {
  abstract subject: string;
  abstract queueGroupName: string;
  abstract onMessage(data: any, msg: Message): void;
  private client: Stan;
  protected ackWait = 5 * 1000;
  constructor(client: Stan) {
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
  listen() {
    const subscription = this.client.subscribe(
      this.subject,
      this.queueGroupName,
      this.subscriptionOptions()
    );
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

Sub class:
``` typescript
class TicketCreatedListener extends Listener {
    subject = 'ticket:created'
    queueGroupName = 'payments-service'
    onMessage(data: any, msg: Message): void {
        console.log('Business Logic Here on the', data);
        msg.ack();
    }
}

```

See [[Enums]] for additional implementation

## Theory: What gets broken out into common modules?

![[Pasted image 20221016164220.png]]
Gives all services the same definitions of all subjects and event interfaces

![[Pasted image 20221016175038.png]]

Defining Events and Event Names in a central location is key to getting multiple microservices working together.

BUT, this approach is limited to an all-TypeScript application. So:
![[Pasted image 20221016175742.png]]



## Theory:  Testing Publishers/Listeners

![[Pasted image 20221016174907.png]]