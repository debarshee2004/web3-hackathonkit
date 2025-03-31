# Setting Up TypeScript Packages: A Senior Engineer's Guide

This guide explains how to properly set up TypeScript packages with a focus on maintainability, best practices, and proper configuration. We'll cover creating a base TypeScript package and then specific configurations for database and types packages.

## Creating a Base TypeScript Package

Start by creating a new directory for your package and initializing it:

```bash
# Create and navigate to your package directory
mkdir package_name && cd package_name

# Initialize with your preferred package manager (using pnpm here)
pnpm init

# Install TypeScript as a dev dependency
pnpm add -D typescript

# Initialize TypeScript configuration
pnpm exec tsc --init

# Optional: Install additional dev dependencies
pnpm add -D @types/node tsup rimraf
```

### Configuring TypeScript

Create or modify your `tsconfig.json` file with proper settings:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
```

### Setting up your package structure

Create the following directories and files:

```bash
mkdir -p src
touch src/index.ts
```

### Configure package.json

Update your `package.json` to correctly define your package:

```json
{
  "name": "package-name",
  "version": "0.1.0",
  "description": "Your package description",
  "main": "dist/index.js",
  "module": "dist/index.mjs",
  "types": "dist/index.d.ts",
  "files": ["dist"],
  "scripts": {
    "build": "tsup src/index.ts --format cjs,esm --dts",
    "clean": "rimraf dist",
    "dev": "tsup src/index.ts --format cjs,esm --dts --watch",
    "lint": "eslint src --ext .ts,.tsx",
    "prepublishOnly": "npm run clean && npm run build"
  },
  "keywords": [],
  "author": "Your Name",
  "license": "MIT",
  "devDependencies": {
    "@types/node": "^20.0.0",
    "eslint": "^8.0.0",
    "rimraf": "^5.0.0",
    "tsup": "^8.0.0",
    "typescript": "^5.8.2"
  },
  "engines": {
    "node": ">=16.0.0"
  }
}
```

## Database Package Setup

When creating a database package with Prisma, follow these steps:

```bash
# Install Prisma dependencies
pnpm add prisma @prisma/client
pnpm add -D @types/node

# Initialize Prisma
npx prisma init
```

### Configure Prisma

Create a `prisma/schema.prisma` file:

```prisma
generator client {
  provider = "prisma-client-js"
  output   = "../node_modules/.prisma/client"
}

datasource db {
  provider = "postgresql" // or any other supported database
  url      = env("DATABASE_URL")
}

// Define your models here
```

### Create database client

Create your database client in `src/index.ts`:

```typescript
import { PrismaClient } from "@prisma/client";

// Create a singleton instance of PrismaClient
export const prismaClient = new PrismaClient({
  log:
    process.env.NODE_ENV === "development"
      ? ["query", "error", "warn"]
      : ["error"],
});

// Add proper error handling
prismaClient.$use(async (params, next) => {
  try {
    return await next(params);
  } catch (error) {
    console.error(`Prisma error in ${params.model}.${params.action}:`, error);
    throw error;
  }
});

// Export types and utilities
export * from "@prisma/client";
export { prismaClient as db };
```

### Update package.json for database package

```json
{
  "name": "database",
  "version": "0.1.0",
  "description": "Database client and utilities",
  "main": "dist/index.js",
  "module": "dist/index.mjs",
  "types": "dist/index.d.ts",
  "files": ["dist", "prisma"],
  "scripts": {
    "build": "tsup src/index.ts --format cjs,esm --dts",
    "clean": "rimraf dist",
    "dev": "tsup src/index.ts --format cjs,esm --dts --watch",
    "db:generate": "prisma generate",
    "db:push": "prisma db push",
    "db:studio": "prisma studio",
    "db:seed": "tsx prisma/seed.ts"
  },
  "exports": {
    ".": {
      "require": "./dist/index.js",
      "import": "./dist/index.mjs",
      "types": "./dist/index.d.ts"
    },
    "./package.json": "./package.json"
  },
  "devDependencies": {
    "@types/node": "^20.0.0",
    "prisma": "^6.5.0",
    "rimraf": "^5.0.0",
    "tsup": "^8.0.0",
    "tsx": "^4.0.0",
    "typescript": "^5.8.2"
  },
  "dependencies": {
    "@prisma/client": "^6.5.0"
  },
  "prisma": {
    "seed": "tsx prisma/seed.ts"
  }
}
```

## Types Package Setup

For a standalone types package:

```bash
mkdir types-package && cd types-package
pnpm init
pnpm add -D typescript
```

### Configure types package

Create a `src/index.ts` file with your types:

```typescript
export interface User {
  id: string;
  name: string;
  email: string;
}

export type UserRole = "admin" | "user" | "guest";

export interface AppConfig {
  apiUrl: string;
  version: string;
  features: {
    darkMode: boolean;
    betaFeatures: boolean;
  };
}
```

### Update package.json for types package

```json
{
  "name": "types",
  "version": "0.1.0",
  "description": "Shared types for the project",
  "main": "dist/index.js",
  "module": "dist/index.mjs",
  "types": "dist/index.d.ts",
  "files": ["dist"],
  "scripts": {
    "build": "tsup src/index.ts --format cjs,esm --dts",
    "clean": "rimraf dist",
    "dev": "tsup src/index.ts --format cjs,esm --dts --watch",
    "prepublishOnly": "npm run clean && npm run build"
  },
  "exports": {
    ".": {
      "require": "./dist/index.js",
      "import": "./dist/index.mjs",
      "types": "./dist/index.d.ts"
    }
  },
  "devDependencies": {
    "rimraf": "^5.0.0",
    "tsup": "^8.0.0",
    "typescript": "^5.8.2"
  }
}
```

## Working with Local Packages

When working with local packages in a monorepo, use the `workspace:*` protocol:

```json
{
  "dependencies": {
    "database": "workspace:*",
    "types": "workspace:*"
  }
}
```

## Best Practices

1. **Use TypeScript Path Aliases**: Configure path aliases in your `tsconfig.json` to make imports cleaner.

2. **Proper Exports**: Use the `exports` field in your `package.json` to control what can be imported from your package.

3. **Provide ESM and CJS Formats**: Use `tsup` to build for both CommonJS and ESM formats.

4. **Versioning**: Follow semantic versioning (semver) for your packages.

5. **Documentation**: Include detailed README files and JSDoc comments for public APIs.

6. **Testing**: Set up Jest or Vitest for comprehensive testing.

7. **CI/CD**: Configure GitHub Actions or similar for continuous integration.

8. **Linting and Formatting**: Use ESLint and Prettier to maintain code quality.

## Troubleshooting Common Issues

### 1. Module Resolution Problems

If you encounter issues with module resolution, check your `tsconfig.json` settings:

```json
{
  "compilerOptions": {
    "moduleResolution": "NodeNext",
    "module": "NodeNext"
  }
}
```

### 2. Publishing Issues

Before publishing, make sure your package is properly built:

```bash
pnpm run build
```

Check your `package.json` to ensure all necessary files are included in the `files` array.

### 3. Type Declaration Files

If your types aren't being recognized by consuming projects, ensure:

1. Your build process is generating `.d.ts` files
2. The `types` field in your `package.json` is correct
3. Type declaration files are included in the published package

By following these guidelines, you'll create well-structured TypeScript packages that are maintainable, properly typed, and easy to use. Remember that the configuration may need adjustments based on your specific project requirements and the ecosystem you're working within.
