One instance of an object/class, passed arround the application. Think the Mongoose connetion.
Use cases:
- Connection Wrappers
```TypeScript
//NatsWrapper.ts
import nats, { Stan } from "node-nats-streaming";
class NatsWrapper {
  private _client?: Stan; // Not creating this until the Constructor is invoked (the instantiation call below)
  // ? tells TS the variable me be undefined at certain times.
  get client() {
    //geter prevents accessing the client before it's called
    if (!this._client) {
      throw new Error("cannot access client before connecting");
    }
    return this._client;
  }
  connect(clusterId: string, clientId: string, url: string) {
    this._client = nats.connect(clusterId, clientId, { url });
    //Promisify the callback so we have async/await available when the connection is called elsewhere
    return new Promise<void>((resolve, reject) => {
      this.client.on("connect", () => {
        console.log("Connected to nats");
        resolve();
      });
      this.client.on("error", (err) => {
        reject(err);
      });
    });
  }
}
export const natsWrapper = new NatsWrapper(); //Invoke, and export.. a singleton instance
```

```TypeScript
//Call in route handler
new TicketCreatedPublisher(natsWrapper.client).publish({
      id: ticket.id,
      price: ticket.price,
      title: ticket.title,
      userId: ticket.userId,
    });
```