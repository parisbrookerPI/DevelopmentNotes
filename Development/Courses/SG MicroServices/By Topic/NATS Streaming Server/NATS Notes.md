## Commnicating Events
- Use node-nats-streaming library

- NATS streaming relies on channels :
`Update event >>> Published to event channel (ticket:updated, ticket:created' etc.) >>> Sent to channel on streaming service >>> Event forwarded to channel subscribers`

- Nats Streaming Server:
	`Channel list`

-  Publisher:
	`Data + Subject (name of chan) ---> stan client ---> Streaming Server`

-  Listener:
	`Streaming Server ---> Subject listened for ---> stan client ---> Subscription`

---
## Queue Groups
- Mechanism in a channel to assign listeners to groups so duplicate services don't do duplicate processing. One member of the queue group is selected randomly, and the message is sent to that member so only it processes the event. (See Lecture 305)

``` TypeScript
// qg is the second argument to the subscribe method
client.on("connect", () => {
  console.log("Listener connected");
  const sub = client.subscribe("ticket:created", "orders-service-queue-group");
  sub.on("message", (msg: Message) => {
    const data = msg.getData();
    if (typeof data === "string") {
      console.log(
        `Recieved msg #${msg.getSequence()} with data ${JSON.parse(data)}`
      );
    }
  });
});

```

## Other Options
Declared with chained method calls rather than an options object:

``` TypeScript
  const options = client
    .subscriptionOptions()
    .setDeliverAllAvailable()
    .setDurableName();
  const sub = client.subscribe("ticket:created", "orders-service-queue-group", options);
```

---
## Manual ACK

Acknowledge the event before allowing it to be dropped

``` typeScript
 const options = client.subscriptionOptions().setManualAckMode(true);
  const sub = client.subscribe(
    "ticket:created",
    "orders-service-queue-group",
    options
  );
```

Must implement manual ACK code in listener

``` TypeScript
sub.on("message", (msg: Message) => {
    const data = msg.getData();
    if (typeof data === "string") {
      console.log(
        `Recieved msg #${msg.getSequence()} with data ${JSON.parse(data)}`
      );
    }
    msg.ack();
  });
```

---
## Client Health Check
- Use the monitoring port set in the K8S depl file:
`kubectl port-forward nats-depl-6f4ff58ffb-6tf5q 8222:8222`

navigate to `http://localhost:8222/streaming`
- `http://localhost:8222/streaming/channelsz?subs=1`

```JSON
{
  "cluster_id": "ticketing",
  "server_id": "hTeRRMGd7iUbs88TzeHYkT",
  "now": "2022-10-16T11:01:13.574499045Z",
  "offset": 0,
  "limit": 1024,
  "count": 1,
  "total": 1,
  "channels": [
    {
      "name": "ticket:created",
      "msgs": 12,
      "bytes": 852,
      "first_seq": 1,
      "last_seq": 12,
      "subscriptions": [
        {
          "client_id": "c0a85f50",
          "inbox": "_INBOX.IBS4FL9CE21NHKGI1G6G5K",
          "ack_inbox": "_INBOX.hTeRRMGd7iUbs88TzeHZCd",
          "queue_name": "orders-service-queue-group",
          "is_durable": false,
          "is_offline": false,
          "max_inflight": 16384,
          "ack_wait": 30,
          "last_sent": 12,
          "pending_count": 0,
          "is_stalled": false
        },
        {
          "client_id": "9bbcda4c",
          "inbox": "_INBOX.1V2KALHNL3YHJXQTNGERVQ",
          "ack_inbox": "_INBOX.hTeRRMGd7iUbs88TzeHZEC",
          "queue_name": "orders-service-queue-group",
          "is_durable": false,
          "is_offline": false,
          "max_inflight": 16384,
          "ack_wait": 30,
          "last_sent": 10,
          "pending_count": 0,
          "is_stalled": false
        }
      ]
    }
  ]
}
```

--- 
## Graceful Client Shutdown
```TypeScript

const client = nats.connect("ticketing", randomBytes(4).toString("hex"), {
  url: "http://127.0.0.1:4222",
});
client.on("connect", () => {
  console.log("Listener connected");
 //Here's the grace
  client.on("close", () => {
    console.log("Nats conn closed");
    process.exit();
  });
  const options = client.subscriptionOptions().setManualAckMode(true);
  const sub = client.subscribe(
    "ticket:created",
    "orders-service-queue-group",
    options
  );
  sub.on("message", (msg: Message) => {
    const data = msg.getData();
    if (typeof data === "string") {
      console.log(
        `Recieved msg #${msg.getSequence()} with data ${JSON.parse(data)}`
      );
    }
    msg.ack();
  });
});

//Here's the grace
process.on("SIGINT", () => client.close());
process.on("SIGTERM", () => client.close());
```

But this isn't failsafe... Message orders can stilll get messed up.

---
## Implementing Durable Queue Groups for Redelivery of Events

- Durable sub stores a record of what events have been processed and not, in the event that unprocessed need to be retrieved in case of service failure
- ![[Pasted image 20221016132830.png]]

```TypeScript
 const options = client
    .subscriptionOptions()
    .setManualAckMode(true)
    .setDeliverAllAvailable() //Still required for first-time startup
    .setDurableName("dursub1");
  const sub = client.subscribe(
    "ticket:created",
    "orders-service-queue-group",
    options
  );

```

 