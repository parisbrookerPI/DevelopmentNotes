----
#### Promisify a callback function to make it awaitable

1) Set up Promise
```TypeScript
// BasePublisher.ts
import { Stan } from "node-nats-streaming";
import { Subjects } from "./Subjects";
interface Event {
  subject: Subjects;
  data: any;
}
export abstract class Publisher<T extends Event> {
  abstract subject: T["subject"];
  private client: Stan;
  constructor(client: Stan) {
    this.client = client;
  }
  publish(data: T["data"]): Promise<void> { //return type
    return new Promise((resolve, reject) => { // Promise created
      this.client.publish(this.subject, JSON.stringify(data), (err) => {
        if (err) { // err first callback pattern
          return reject(err);
        }
        resolve();
      });
    });
  }
}

```

2) Actually call the method
```typescript
stan.on("connect", async () => {
  console.log("Publisher connected to NATS");
  const publisher = new TicketCreatedPublisher(stan);
  try {
    await publisher.publish({
      id: "gh3744",
      price: 33,
      title: "Metallica",
    });
  } catch (err) {
    console.error(err);
  }
});

```

-----
