# types/

Shared TypeScript types and interfaces used across the app.

## What goes here
- Custom types that multiple files share
- Extending built-in types (e.g. adding `user` to Express `Request`)
- Enums and constants with types

## Example

**index.ts** — shared interfaces
```typescript
export interface ApiResponse<T> {
  success: boolean;
  data: T;
  message?: string;
}

export interface PaginatedResult<T> {
  items: T[];
  total: number;
  page: number;
  limit: number;
}

export enum UserRole {
  ADMIN = 'admin',
  USER  = 'user',
}
```

**express.d.ts** — extend Express Request to include the logged-in user
```typescript
import { JwtPayload } from 'jsonwebtoken';

declare global {
  namespace Express {
    interface Request {
      user?: JwtPayload;
    }
  }
}
```

Without this, TypeScript throws an error when you try to access `req.user` in your middleware.
