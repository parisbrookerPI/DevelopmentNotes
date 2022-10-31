Go copy a cookie and see what it looks like  at base64decode.org

``` typescript
//Helper function to get cokkie for other tests
global.signin = async () => {
  // Build a jwt payload {id, email }
  // Create the jwt! (using the key)
  // Build the session object - take the jwt and put it in an {jwt: myJwt}
  // Turn session object into {}
  // Encode session in base 64
  // Return string of encoded data
};
```

One small fix is required to return the cookie to prevent our tests from failing:
Find the **return** of the **global.signin** method and change this:
  ``return [`express:sess=${base64}`];``
to this:
  ``return [`**session**=${base64}`];``

```typescript

global.signin = async () => {
  // Build a jwt payload {id, email }
  const payload = {
    id: "dsdkskdsd",
    email: "test@test.com",
  };
  // Create the jwt! (using the key)
  const token = jwt.sign(payload, process.env.JWT_KEY!);
  // Build the session object - take the jwt and put it in an {jwt: myJwt}
  const session = { jwt: token };
  // Turn session object into {}
  const sessionJSON = JSON.stringify(session);
  // Encode session in base 64
  const base64 = Buffer.from(sessionJSON).toString("base64");
  // Return string of encoded data
  return [`**session**=${base64}`];
};

```

``` Typescript
declare global {
  var signin: () => Promise<string[]>;
}
## BECOMES
declare global {
  var signin: () => string[];
}

#AND REMOVE ASYNC FROM GLOBAL FUNCTION AS IT IS NOT A PROMISE ANYMORE
```
