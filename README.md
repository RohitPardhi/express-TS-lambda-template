# Express + TypeScript — AWS Lambda Setup Guide

## Prerequisites
- Node.js installed (v18+ recommended)
- npm or yarn
- AWS CLI configured (`aws configure`)

---

## Step 1 — Initialize the project

```bash
mkdir my-lambda-app && cd my-lambda-app
npm init -y
```

---

## Step 2 — Install dependencies

```bash
# Runtime dependencies
npm install express dotenv aws-serverless-express @vendia/serverless-express

# Dev dependencies
npm install -D typescript ts-node nodemon @types/node @types/express @types/aws-lambda
```

> **Note:** `@vendia/serverless-express` is the actively maintained fork of `aws-serverless-express`. It wraps your Express app so Lambda can invoke it like a regular HTTP handler.

---

## Step 3 — Initialize TypeScript

```bash
npx tsc --init
```

Update your `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "rootDir": "src",
    "outDir": "dist",
    "strict": true,
    "esModuleInterop": true,
    "resolveJsonModule": true,
    "skipLibCheck": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

---

## Step 4 — Create folder structure

```bash
mkdir -p src/{config,controllers,middleware,models,routes,services,types,utils}
```

---

## Step 5 — Create the Express app

**`src/app.ts`** — Pure Express app, no `listen()` call here.

```typescript
import express, { Application } from 'express';
import cors from 'cors';

const app: Application = express();

// Middleware
app.use(cors());
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true, limit: '10mb' }));

// Routes
app.get('/health', (req, res) => {
  res.json({ status: 'OK' });
});

export default app;
```

**`src/lambda.ts`** — Lambda entry point. This is what AWS invokes.

```typescript
import serverlessExpress from '@vendia/serverless-express';
import app from './app';

export const handler = serverlessExpress({ app });
```

**`src/index.ts`** — Local dev server only. Not used in Lambda.

```typescript
import dotenv from 'dotenv';
dotenv.config();

import app from './app';

const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
  console.log(`Local server running on http://localhost:${PORT}`);
});
```

---

## Step 6 — Create .env file

```bash
touch .env
```

```env
PORT=3000
NODE_ENV=development
```

> **Lambda env vars** are set separately in the Lambda console or via your deployment config — not from `.env`.

---

## Step 7 — Configure nodemon (local dev only)

Create `nodemon.json` in the root:

```json
{
  "watch": ["src"],
  "ext": "ts",
  "ignore": ["src/**/*.spec.ts"],
  "exec": "ts-node src/index.ts"
}
```

---

## Step 8 — Add scripts to package.json

```json
"scripts": {
  "dev": "nodemon",
  "build": "tsc",
  "start": "node dist/index.js",
  "package": "npm run build && node scripts/package.js",
  "lint": "eslint src/**/*.ts"
}
```

---

## Step 9 — Run locally

```bash
npm run dev
```

Visit `http://localhost:3000/health` — you should see `{ "status": "OK" }`.

---

## Step 10 — Build and package for Lambda

### 10a — Compile TypeScript

```bash
npm run build
```

This outputs compiled JS to `/dist`.

### 10b — Install only production dependencies into a deployment folder

```bash
mkdir -p lambda-package
cp -r dist lambda-package/
cp package.json lambda-package/
cd lambda-package && npm install --omit=dev
```

### 10c — Zip the package

```bash
cd lambda-package
zip -r ../function.zip .
```

### 10d — Deploy to Lambda

```bash
aws lambda update-function-code \
  --function-name your-function-name \
  --zip-file fileb://../function.zip
```

> **Lambda handler config:** In your Lambda function settings, set the handler to `dist/lambda.handler`.

---

## Step 11 — Configure API Gateway

Lambda alone doesn't expose an HTTP endpoint — you need API Gateway in front of it.

1. Go to **API Gateway** → Create **HTTP API** (v2, simpler and cheaper than REST API).
2. Add a **Lambda integration** pointing to your function.
3. Set route to `$default` (catches all methods and paths, letting Express handle routing).
4. Deploy the API — you'll get an invoke URL like:
   ```
   https://abc123.execute-api.ap-south-1.amazonaws.com
   ```

---

## Lambda-specific considerations

| Topic | Detail |
|---|---|
| **Cold starts** | First invocation after idle spins up a new container. Keep `node_modules` lean. |
| **Timeout** | Default is 3s — increase to 30s+ for routes doing heavy work (image processing, DB calls). |
| **Payload size** | API Gateway HTTP API limit is **10MB** for requests/responses. Use S3 pre-signed URLs for large file uploads instead of base64 in the body. |
| **Stateless** | No in-memory state persists between invocations. Use a DB or S3 for persistent data. |
| **Environment variables** | Set in Lambda console → Configuration → Environment variables. Never commit secrets. |
| **IAM permissions** | Attach an IAM role to the Lambda function. Grant only what it needs (e.g., `s3:PutObject` for a specific bucket). |
| **Lambda Layers** | Use Layers to share large `node_modules` (like native binaries) across functions and keep your deployment zip small. |

---

## Optional — Lambda Layers for node_modules

If your `node_modules` are large (>50MB unzipped), extract them into a Layer:

```bash
mkdir -p layer/nodejs
cp -r node_modules layer/nodejs/
cd layer && zip -r ../layer.zip nodejs/

aws lambda publish-layer-version \
  --layer-name my-app-deps \
  --zip-file fileb://../layer.zip \
  --compatible-runtimes nodejs18.x nodejs20.x
```

Then attach the layer ARN to your Lambda function and exclude `node_modules` from your deployment zip.

---

## Optional — Add ESLint + Prettier

```bash
npm install -D eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin prettier eslint-config-prettier
```

Create `.eslintrc.json`:

```json
{
  "parser": "@typescript-eslint/parser",
  "plugins": ["@typescript-eslint"],
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "prettier"
  ],
  "env": {
    "node": true,
    "es2020": true
  }
}
```

Create `.prettierrc`:

```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5"
}
```

---

## Final structure

```
my-lambda-app/
├── src/
│   ├── config/
│   ├── controllers/
│   ├── middleware/
│   ├── models/
│   ├── routes/
│   ├── services/
│   ├── types/
│   ├── utils/
│   ├── app.ts           ← Express app (no listen)
│   ├── lambda.ts        ← Lambda handler entry point
│   └── index.ts         ← Local dev server only
├── dist/                ← generated after build
├── lambda-package/      ← deployment staging folder
├── function.zip         ← upload this to Lambda
├── .env                 ← local dev only
├── .env.example
├── .eslintrc.json
├── .gitignore
├── nodemon.json
├── package.json
├── tsconfig.json
└── README.md
```

---

## Quick commands cheat sheet

| Command | What it does |
|---|---|
| `npm run dev` | Start local dev server with hot reload |
| `npm run build` | Compile TS to JS in `/dist` |
| `npm start` | Run compiled build locally |
| `npm run lint` | Check for lint errors |
| `zip -r function.zip lambda-package/` | Package for Lambda deployment |
| `aws lambda update-function-code ...` | Deploy zip to Lambda |
