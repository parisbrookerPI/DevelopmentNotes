```TypeScript
// @ts-nocheck
import { Order } from "./Order";
// To associate an existing Order and Ticket together
const ticket = await Ticket.findOne({});
const order = await Order.findOne({});
order.ticket = ticket;
await order.save();
// To associate an existing ticket with a new order
const ticket = await Ticket.findOne({});
const order = Order.build({
  ticket: ticket,
  userId: "...",
  status: OrderStatus.Created,
  expiresAt: someDate,
});
// To fetch an existing order from the database with the associated ticket.
const order = await Order.findById("..").populate("ticket");
```