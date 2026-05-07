# utils/

Small, reusable helper functions that don't belong anywhere else.

## What goes here
- Formatting helpers (dates, strings, numbers)
- Reusable validators
- Logger setup
- Anything used in more than one place

## Example

**logger.ts** — a simple logger
```typescript
export const logger = {
  info:  (msg: string) => console.log(`[INFO]  ${new Date().toISOString()} — ${msg}`),
  error: (msg: string) => console.error(`[ERROR] ${new Date().toISOString()} — ${msg}`),
};
```

**helpers.ts** — small reusable functions
```typescript
// Capitalize first letter of a string
export const capitalize = (str: string): string =>
  str.charAt(0).toUpperCase() + str.slice(1);

// Check if a value is a valid email
export const isValidEmail = (email: string): boolean =>
  /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);

// Sleep for N milliseconds (useful for retries)
export const sleep = (ms: number): Promise<void> =>
  new Promise(resolve => setTimeout(resolve, ms));
```

If you find yourself writing the same small function in multiple files, it belongs here.
