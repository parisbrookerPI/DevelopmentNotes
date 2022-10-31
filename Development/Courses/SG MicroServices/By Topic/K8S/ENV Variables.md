```YAML
    spec:
      containers:
        - name: tickets
          image: paris2083/tickets
          env:
            - name: MONGO_URI
              value: "mongodb://tickets-mongo-srv:27017/tickets"
            - name: JWT_KEY
              valueFrom:
                secretKeyRef:
                  name: jwt-secret
                  key: JWT_KEY
```

## Secret Variables
`kubectl create secret generic jwt-secret --from-literal JWT_KEY=asdf`

## Dynamic Variables
```YAML
      containers:
        - name: tickets
          image: paris2083/tickets
          env:
            - name:  NATS_CLIENT_ID
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: MONGO_URI
```