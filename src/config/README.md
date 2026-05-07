# config/

App-wide configuration and setup files.

## What goes here
- Environment variable parsing
- Database connection setup
- Third-party service config (Redis, S3, etc.)

## Example

**env.ts** — parse and export all env variables in one place
```typescript
export const config = {
  port: process.env.PORT || 3000,
  dbUrl: process.env.DATABASE_URL || '',
  jwtSecret: process.env.JWT_SECRET || '',
};
```

**db.ts** — setup your database connection
```typescript
import mongoose from 'mongoose';
import { config } from './env';

export const connectDB = async () => {
  await mongoose.connect(config.dbUrl);
  console.log('DB connected');
};
```

Instead of scattering `process.env.PORT` everywhere, you import from config once.
