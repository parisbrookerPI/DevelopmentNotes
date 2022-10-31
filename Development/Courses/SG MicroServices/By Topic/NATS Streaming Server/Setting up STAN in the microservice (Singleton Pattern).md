The snag in just putting it in the index file

Mongoose natively keeps track of the connections made throughout the application. NATS doesn't do this. Calling the NATS client creates a cilent instantly. Danger of circular dependency:

![[Pasted image 20221017174434.png]]

A better, singleton, approach:

![[Pasted image 20221017181052.png]]

```TypeScript

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

Notice the graceful shutdown logic is in the index file, not the wrapper hiden away; bad design to hid somethign that can close the whole program