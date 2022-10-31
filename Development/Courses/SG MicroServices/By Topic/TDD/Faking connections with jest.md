We don't really want to run a nats server for testing, so need some way of mocking the connection.

![[Pasted image 20221021195607.png]]

see `__mocks__` for implementation

![[Pasted image 20221022063435.png]]

```Typescript
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

New only cares about `client: Stan` but waht does it do?:

![[Pasted image 20221022063859.png]]

but what about the base publisher class?
![[Pasted image 20221022064015.png]]

 ```Typescript
 export const natsWrapper = {
  client: {
    // publish: (subject: string, data: string, callback: () => void) => {
    //   callback(); Fake implementation
    // },
    publish: jest
      .fn()
      .mockImplementation(
        (subject: string, data: string, callback: () => void) => {
          callback();
        }
      ),
  },
};
```

See new.ts and update.ts for implementation of `jest.fn()` 