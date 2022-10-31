# Setting up for TDD
```
E:.
│   app.ts
│   tickets.ts
│
├───routes
│   └───__test__
│           new.test.ts
│
└───test
        setup.ts
```

setup.ts
```TypeScript
import request from "supertest";
import { MongoMemoryServer } from "mongodb-memory-server";
import mongoose from "mongoose";
import { app } from "../app";
// declare global {
//   namespace NodeJS {
//     export interface Global {
//       signin(): Promise<string[]>;
//     }
//   }
// }
declare global {
  var signin: () => Promise<string[]>;
}
let mongo: any;
beforeAll(async () => {
  mongo = await MongoMemoryServer.create();
  const mongoUri = mongo.getUri();
  await mongoose.connect(mongoUri, {});
});
beforeEach(async () => {
  process.env.JWT_KEY = "asdf";
  const collections = await mongoose.connection.db.collections();
  for (let collection of collections) {
    await collection.deleteMany({});
  }
});
afterAll(async () => {
  if (mongo) {
    await mongo.stop();
  }
  await mongoose.connection.close();
});
//Helper function to get cokkie for other tests
global.signin = async () => {
  const email = "test1@test.com";
  const password = "password";
  const response = await request(app)
    .post("/api/users/signup")
    .send({ email, password })
    .expect(201);
  const cookie = response.get("Set-Cookie");
  return cookie;
};
// change to:
//     declare global {
//       var signin: () => Promise<string[]>;
//     }
```


new.test.ts
``` Typescript
import request from "supertest";
import { app } from "../../app";

  
it('')
it('')
it('')
it('')
```
# What is being tested?
### Routes
- Route handler present
- Validate request body (presence and type)
- Authenticated
-  Desired return is provided
```TypeScript
it("has a route handler listening to /api/tickets for post requests", async () => {});
it("can only be accessed if user is signed in", async () => {});
it("returns error if invalid title", async () => {});
it("returns error if invalid price", async () => {});
it("creates a valid ticket", async () => {});

```
