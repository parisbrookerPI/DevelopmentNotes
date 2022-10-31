
## Define Interfaces

Properties required to buiold a new ticket
| TicketAttrs |        |
| ----------- | ------ |
| title       | string |
| price       | number |
| userId      | sring  |

Properties that a ticket has (inc those auto added - may be different to attrs )
| TicketDoc |        |
| --------- | ------ |
| title     | string |
| price     | number |
| userId    | string |
| createAt  | string |

The actual built document
| TicketModel |                |
| ----------- | -------------- |
| build       | (attrs) => Doc |


## Write Code Template

``` TypeScript
import mongoose from "mongoose";

// core attrs
interface TicketAttrs {
    title: string
    price: number
    userId: string
}
// Gives us the opportunity to add attributes later
interface TicketDoc extends mongoose.Document {
    title: string
    price: number
    userId: string
}
// the build function that allows TypeScript to be happy
interface TicketModel extends mongoose.Model<TicketDoc>{
    build(attrs: TicketAttrs) : TicketDoc
}

// The mongoose document schema itself
const ticketSchema = new mongoose.Schema({

})

```

## Implement Code

```Typescript
import mongoose from "mongoose";
// core attrs
interface TicketAttrs {
  title: string;
  price: number;
  userId: string;
}
// Gives us the opportunity to add attributes later
interface TicketDoc extends mongoose.Document {
  title: string;
  price: number;
  userId: string;
}
// the build function that allows TypeScript to be happy
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
    },
    userId: {
      type: String,
      required: true,
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
const Ticket = mongoose.model<TicketDoc, TicketModel>("Ticket", ticketSchema);
export { Ticket };
```