# controllers/

Handles incoming HTTP requests and sends responses. Nothing more.

## What goes here
- Read data from `req` (body, params, query)
- Call a service to do the actual work
- Send back a response via `res`

## Example

**user.controller.ts**
```typescript
import { Request, Response } from 'express';
import { UserService } from '../services/user.service';

export const getUser = async (req: Request, res: Response) => {
  const user = await UserService.findById(req.params.id);
  res.json(user);
};

export const createUser = async (req: Request, res: Response) => {
  const user = await UserService.create(req.body);
  res.status(201).json(user);
};
```

Controllers are thin — they don't contain business logic. They just pass data to services and return the result.
