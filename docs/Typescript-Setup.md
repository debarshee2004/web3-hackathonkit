# Creating a Typescript Package

Setting up the package with Typescript.

```bash
mkdir package_name && cd package_name

pnpm init
pnpm add -D typescript

pnpm tsc --init
pnpm tsc -w
```

## Database Package

```bash
pnpm install prisma @prisma/client
npx prisma init
```

```ts
// src/index.ts
import { PrismaClient } from "@prisma/client";

export const prismaClient = new PrismaClient();
```

```json
// package.json
{
  "name": "database",
  "version": "1.0.0",
  "description": "",
  "main": "./src/index.ts",
  "type": "module",
  "exports": {
    "./dbclient": "./src/index.ts"
  },
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "typescript": "^5.8.2"
  },
  "dependencies": {
    "@prisma/client": "^6.5.0",
    "prisma": "^6.5.0"
  },
  "prisma": {
    "seed": "./prisma/seed.ts"
  }
}
```

## Types Package

```json
// package.json
{
  "name": "types",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "exports": {
    "./types": "./index.ts"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "typescript": "^5.8.2"
  }
}
```
