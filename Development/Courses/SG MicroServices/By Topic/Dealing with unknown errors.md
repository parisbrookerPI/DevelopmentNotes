Current errorhandler:

``` typescript
import { Request, Response, NextFunction } from "express";
import { CustomError } from "../errors/customerror";
//4 arguments makes this an error handler
export const errorHandler = (
  err: Error,
  req: Request,
  res: Response,
  next: NextFunction
) => {
  if (err instanceof CustomError) {
    return res.status(err.statusCode).send({ errors: err.seraliseErrors() });
  }
// Need to implement somethinghere that logs the error if not caught by CustomError
console.log(err)

  res.status(400).send({
    errors: [{ message: "something went wrong" }],
  });
};
```

