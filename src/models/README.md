# models/

Defines the shape of your data and how it maps to the database.

## What goes here
- Database schemas (Mongoose, Prisma, TypeORM, etc.)
- Type definitions that match your DB tables/collections

## Example (Mongoose)

**user.model.ts**
```typescript
import mongoose, { Schema, Document } from 'mongoose';

export interface IUser extends Document {
  name: string;
  email: string;
  password: string;
  createdAt: Date;
}

const UserSchema = new Schema<IUser>({
  name:      { type: String, required: true },
  email:     { type: String, required: true, unique: true },
  password:  { type: String, required: true },
  createdAt: { type: Date, default: Date.now },
});

export const User = mongoose.model<IUser>('User', UserSchema);
```

Think of models as the blueprint — they define what a User, Post, or Product looks like in your database.
