![[Pasted image 20221022140010.png]]

![[Pasted image 20221022134126.png]]

![[Pasted image 20221022134148.png]]

```TypeScript
//OrderStatus.ts
export enum OrderStatus {
  // When order is created but ticket has not been reserved
  Created = "created",
  // When a ticket has already been reserved, order expires, or the user cancels the order
  Cancelled = "cancelled",
  // Order has reserved ticket
  AwaitingPayment = "awaiting:payment",
  // payment has been completed after reservation
  Complete = "complete",
}
```

```Typescript
//Order.ts

import mongoose from "mongoose";
import { OrderStatus } from "@parisbtickets/common";
import { TicketDoc } from "./Ticket";
interface OrderAttrs {
  userId: string;
  status: OrderStatus;
  expiresAt: Date;
  ticket: TicketDoc;
}
// End of 366 for this
interface OrderDoc extends mongoose.Document {
  userId: string;
  status: OrderStatus;
  expiresAt: Date;
  ticket: TicketDoc;
}
interface OrderModel extends mongoose.Model<OrderDoc> {
  build(attrs: OrderAttrs): OrderDoc;
}
const orderSchema = new mongoose.Schema(
  {
    userId: {
      type: String,
      required: true,
    },
    status: {
      type: String,
      required: true,
      enum: Object.values(OrderStatus),
      defualt: OrderStatus.Created,
    },
    expiresAt: {
      type: mongoose.Schema.Types.Date,
    },
    ticket: {
      type: mongoose.Schema.Types.ObjectId,
      ref: "Ticket",
    },
  },
  {
    toJSON: {
      transform(doc, ret) {
        ret.id = ret._id;
        delete ret._id;
      },
    },
  }
);
orderSchema.statics.build = (attrs: OrderAttrs) => {
  return new Order(attrs);
};
const Order = mongoose.model<OrderDoc, OrderModel>("Orders", orderSchema);
export { Order };
```

```TypeScript
//Ticket.ts
import mongoose from "mongoose";
interface TicketAttrs {
  title: string;
  price: number;
}
export interface TicketDoc {
  title: string;
  price: number;
}
interface TicketModel extends mongoose.Model<TicketDoc> {
  build(attrs: TicketAttrs): TicketDoc;
}
const ticketSchema = new mongoose.Schema(
  {
    title: {
      type: String,
      required: true,
    },
    price: {
      type: Number,
      required: true,
      min: 0,
    },
  },
  {
    toJSON: {
      transform(doc, ret) {
        ret.id = ret._id;
        delete ret._id;
      },
    },
  }
);
ticketSchema.statics.build = (attrs: TicketAttrs) => {
  return new Ticket(attrs);
};
const Ticket = mongoose.model<TicketDoc, TicketModel>("Tickets", ticketSchema);
export { Ticket };
```