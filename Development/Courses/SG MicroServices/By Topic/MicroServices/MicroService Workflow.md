
1) Create package.json, install dependencies
2) Write Dockerfile
3) Create index.ts
4) Build docker, push to dockerhub
5) Write K8s for deployemnet and service
6) Update skaffold.yaml
7) Wrtie k8s for mongo deployment and service 


## Define Routes and Data Structure

| ROUTE            | METHOD | BODY                         | GOAL                     | CONSIDERATIONS | 
| ---------------- | ------ | ---------------------------- | ------------------------ | -------------- |
| /apit/tickets    | GET    | -                            | Retrieve all tickets     |                |
| /api/tickets/:id | GET    | -                            | Retrieve specific ticket |                |
| /api/tickets     | POST   | {title:string, price:string} | Create a ticket          |                |
| /api/tickets/:id | PUT    | {title:string, price:string} | Update a ticket          | Owner, check id                |

## Implement Tests, then Implement Routes, MiddleWare and Business Logic

## Expose API through nginx
