# services/

Contains the actual business logic of your app.

## What goes here
- Database queries (create, find, update, delete)
- Business rules and calculations
- Calls to external APIs

## Example

**user.service.ts**
```typescript
import { User } from '../models/user.model';
import bcrypt from 'bcrypt';

export const UserService = {
  findById: async (id: string) => {
    return await User.findById(id);
  },

  create: async (data: { name: string; email: string; password: string }) => {
    const hashed = await bcrypt.hash(data.password, 10);
    return await User.create({ ...data, password: hashed });
  },

  delete: async (id: string) => {
    return await User.findByIdAndDelete(id);
  },
};
```

Services are the brain. Controllers ask "what should I do?" — services actually do it.
