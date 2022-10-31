**Lecture 441
#redis
#bulljs

---
# Overview

What is the best way to provide an expiration service?
Of all the options:
- Native event bus feature to delay an expiration:complete (scheduled event) - NOT AVAILABLE IN NATS
- Redis Server and bull.js

![[Pasted image 20221031171648.png]]

## Technology
- Bull.js
- Redis

## Theroy Points

-  Job processing and scheduling
- 
---
# Initial Setup

```
E:.
│   .dockerignore
│   Dockerfile
│   package.json
│   tsconfig.json
└───src
    │   expiration.ts
    │   NatsWrapper.ts
    └───__mocks__
            NatsWrapper.ts
```

### Docker
```docker
FROM node:alpine
WORKDIR /app
COPY package.json .
RUN npm install --only=prod
COPY . .
CMD ["npm", "start"]
```

### Redis

```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: expiration-redis-depl
spec:
  replicas: 1
  selector:
    matchLabels:
      app: expiration-redis
  template:
    metadata:
      labels:
        app: expiration-redis
    spec:
      containers:
        - name: expiration-redis
          image: redis
---
apiVersion: v1
kind: Service
metadata:
  name: expiration-redis-srv
spec:
  selector:
    app: expiration-redis
  ports:
    - name: db
      protocol: TCP
      port: 6379
      targetPort: 6379
```

### Expiration - just depl, no srv

```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: expiration-depl
spec:
  replicas: 1
  selector:
    matchLabels:
      app: expiration
  template:
    metadata:
      labels:
        app: expiration
    spec:
      containers:
        - name: expiration
          image: paris2083/expiration
          env:
            - name: NATS_CLIENT_ID
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: NATS_URL
              value: "http:/nats-srv:4222"
            - name: CLUSTER_ID
              value: ticketing
            - name: REDIS_HOST
              value: expiration-redis-srv
```

---
# Listener Setup

```TypeScript
//filename.ts
import {
  Listener,
  OrderCreatedEvent,
  OrderStatus,
  Subjects,
} from "@parisbtickets/common";
import { Message } from "node-nats-streaming";
import { queueGroupName } from "./QueueGroupName";
export class OrderCreatedListener extends Listener<OrderCreatedEvent> {
  readonly subject = Subjects.OrderCreated;
  queueGroupName = queueGroupName;
  async onMessage(data: OrderCreatedEvent["data"], msg: Message) {}
}
```

---
# Bull JS

Classic implementation of bull
![[Pasted image 20221031183037.png]]

Bull isn't really for messaging, more for JobScheduling

**Terminology**
queue - level of abstraction

In current app:

![[Pasted image 20221031183336.png]]

```TypeScript
//filename.ts
{}
```

---
# Testing

```TypeScript
//filename.ts
{}
```

---
# Implementation Step

## Iteration?

```TypeScript
//filename.ts
{}
```

---
# Testing

```TypeScript
//filename.ts
{}
```

---
# Procedural Comments

1)  Step
2)  Step
3)  step**Lecture/Chapter/Page references**
#TopicHashTags
#TopicHashTags
#TopicHashTags

---
# Overview

## Technology

## Application-Specific

## Theroy Points

---
# Implementation Step

## Iteration?

```TypeScript
//filename.ts
{}
```

---
# Testing

```TypeScript
//filename.ts
{}
```

---
# Implementation Step

## Iteration?

```TypeScript
//filename.ts
{}
```

---
# Testing

```TypeScript
//filename.ts
{}
```

---
# Procedural Comments

1)  Step
2)  Step
3)  step**Lecture/Chapter/Page references**
#TopicHashTags
#TopicHashTags
#TopicHashTags

---
# Overview

## Technology

## Application-Specific

## Theroy Points

---
# Implementation Step

## Iteration?

```TypeScript
//filename.ts
{}
```

---
# Testing

```TypeScript
//filename.ts
{}
```

---
# Implementation Step

## Iteration?

```TypeScript
//filename.ts
{}
```

---
# Testing

```TypeScript
//filename.ts
{}
```

---
# Procedural Comments

1)  Step
2)  Step
3)  step**Lecture/Chapter/Page references**
#TopicHashTags
#TopicHashTags
#TopicHashTags

---
# Overview

## Technology

## Application-Specific

## Theroy Points

---
# Implementation Step

## Iteration?

```TypeScript
//filename.ts
{}
```

---
# Testing

```TypeScript
//filename.ts
{}
```

---
# Implementation Step

## Iteration?

```TypeScript
//filename.ts
{}
```

---
# Testing

```TypeScript
//filename.ts
{}
```

---
# Procedural Comments

1)  Step
2)  Step
3)  step**Lecture/Chapter/Page references**
#TopicHashTags
#TopicHashTags
#TopicHashTags

---
# Overview

## Technology

## Application-Specific

## Theroy Points

---
# Implementation Step

## Iteration?

```TypeScript
//filename.ts
{}
```

---
# Testing

```TypeScript
//filename.ts
{}
```

---
# Implementation Step

## Iteration?

```TypeScript
//filename.ts
{}
```

---
# Testing

```TypeScript
//filename.ts
{}
```

---
# Procedural Comments

1)  Step
2)  Step
3)  step