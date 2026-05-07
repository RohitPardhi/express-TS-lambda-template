# routes/

Maps URLs to the right controller functions.

## What goes here
- Route definitions (GET, POST, PUT, DELETE)
- Attach middleware to specific routes
- One file per resource (users, posts, products, etc.)

## Example

**user.routes.ts**
```typescript
import { Router } from 'express';
import { getUser, createUser } from '../controllers/user.controller';
import { authMiddleware } from '../middleware/auth.middleware';

const router = Router();

router.get('/:id', authMiddleware, getUser);
router.post('/', createUser);

export default router;
```

**index.ts** — combine all routes in one place
```typescript
import { Application } from 'express';
import userRoutes from './user.routes';

export const registerRoutes = (app: Application) => {
  app.use('/api/users', userRoutes);
};
```

Routes are just a map — they don't contain logic, they just point to the right controller.
