# middleware/

Functions that run between the request and the controller.

## What goes here
- Authentication / authorization checks
- Request validation
- Logging
- Error handling

## Example

**auth.middleware.ts** — block requests without a valid token
```typescript
import { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';

export const authMiddleware = (req: Request, res: Response, next: NextFunction) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ message: 'Unauthorized' });

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET!);
    req.user = decoded;
    next(); // pass to the next handler
  } catch {
    res.status(401).json({ message: 'Invalid token' });
  }
};
```

**error.middleware.ts** — catch all unhandled errors
```typescript
import { Request, Response, NextFunction } from 'express';

export const errorHandler = (err: Error, req: Request, res: Response, next: NextFunction) => {
  console.error(err.message);
  res.status(500).json({ message: 'Something went wrong' });
};
```
