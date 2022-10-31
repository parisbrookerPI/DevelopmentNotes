``` TypeScript
import mongoose from "mongoose";
import { Password } from "../services/Password";
// An interface that describes properties required to create a new user
interface UserAttrs {
  email: string;
  password: string;
}
// Interface descrbing User Model properties
interface UserModel extends mongoose.Model<UserDoc> {
  build(attrs: UserAttrs): UserDoc;
}
//Interface to describe what properties a User Document has (solves issue 2)
interface UserDoc extends mongoose.Document {
  email: string;
  password: string;
}
const userSchema = new mongoose.Schema(
  {
    email: {
      type: String,
      required: true,
    },
    password: {
      type: String,
      required: true,
    },
  },
  {
    toJSON: {
      //Mongoose feature to allow us to change the returned object
      transform(doc, ret) {
        delete ret.password;
        delete ret.__v;
        ret.id = ret._id;
        delete ret._id;
      },
    },
  }
);
//Mongoose middlesware - when using async, need to pass in done and then call done at end of function
// Don't use arrow function as need access to this property.
userSchema.pre("save", async function (done) {
  if (this.isModified("password")) {
    const hashed = await Password.toHash(this.get("password"));
    this.set("password", hashed);
  }
  done();
});
//Adds custom method onto the Class/Model - Allowed TS to do some checking
userSchema.statics.build = (attrs: UserAttrs) => {
  return new User(attrs);
};
const User = mongoose.model<UserDoc, UserModel>("User", userSchema);
// new User({
//   email: "test@test.com",
//   password: "asdiofjio",
// });
// Basicallly a wrapper for the mongoose constructor which introduces a Type checking stage (Solves Issue 1)
// HOWEVER, means that two functions need to be exported... new User.build would be better...
// SO... add a static method to the Schema.. but then, TS won't recognise the ew method... so, another interface (see top)
// buildUser({
//   email: "asdasd@asdifas.dco",
//   password: "sadfkasd",
// });
export { User };
```
