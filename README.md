# PWAExempleAuth
Exemple de PWA avec Auth.js

## Outils

### [GitHub](https://docs.github.com/en)
### [Next.js](https://nextjs.org/docs)
### [Vercel](https://vercel.com/docs)
### [Prisma](https://www.prisma.io/docs)
### [PostgreSQL](https://www.postgresql.org/docs/)
### [Auth.js](https://authjs.dev/getting-started)
### [Resend](https://resend.com/docs/dashboard/domains/introduction)

## Step 1 - Git clone
Dans la console:
git clone https://github.com/KenCacciabueOrif/PWAExempleAuth.git   

## Step 2 - Create Next app

### 2.1
Dans la console:
npx create-next-app@latest

### 2.2
Enlever les contenus inutiles

## Step 3 - Importer sur Vercel

## Step 4 - Installer Auth.js

### 4.1
Dans la console:
npm install next-auth@beta

### 4.2
Dans la console:
npx auth secret

### 4.3
Créé un fichier auth.ts à la racine avec le code suivant

```
import NextAuth from "next-auth"
 
export const { handlers, signIn, signOut, auth } = NextAuth({
  providers: [],
})
```

### 4.4
Créé un fichier ./app/api/auth/[...nextauth]/route.ts avec le code suivant

```
import { handlers } from "@/auth" // Referring to the auth.ts we just created
export const { GET, POST } = handlers
```

### 4.5
Ajouter un fichier middleware.ts à la racine avec le code suivant
```
export { auth as middleware } from "@/auth"
```

## Step 5 - Installer prisma et son adapter

### 5.1
Dans la console:
npm install @prisma/extension-accelerate @prisma/client @auth/prisma-adapter
### 5.2
Dans la console:
npm install prisma tsx --save-dev

### 5.3
Dans la console:
npx prisma init --db --output ../app/generated/prisma

### 5.4
Créé le fichier lib/prisma.ts avec le code suivant:

```
import { PrismaClient } from "@prisma/client"
import { withAccelerate } from "@prisma/extension-accelerate";
 
const globalForPrisma = globalThis as unknown as { prisma: PrismaClient }
 
export const prisma = globalForPrisma.prisma || new PrismaClient().$extends(withAccelerate())
 
if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma

export default prisma;
```

### 5.5
Modifier tsconfig.ts de la manière suivante:
```
"paths": {
      "@/*": ["./*"],
      "@/lib/*": ["./lib/*"]
    }
```
### 5.6
Modifier prisma/schema de la manière suivante:
```
// This is your Prisma schema file,
// learn more about it in the docs: https://pris.ly/d/prisma-schema

// Looking for ways to speed up your queries, or scale easily with your serverless or edge functions?
// Try Prisma Accelerate: https://pris.ly/cli/accelerate-init

generator client {
  provider = "prisma-client-js"
  output   = "../app/generated/prisma"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id            String          @id @default(cuid())
  name          String?
  email         String          @unique
  emailVerified DateTime?
  image         String?
  accounts      Account[]
  sessions      Session[]
 
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
 
model Account {
  userId            String
  type              String
  provider          String
  providerAccountId String
  refresh_token     String?
  access_token      String?
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String?
  session_state     String?
 
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
 
  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
 
  @@id([provider, providerAccountId])
}
 
model Session {
  sessionToken String   @unique
  userId       String
  expires      DateTime
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)
 
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
 
model VerificationToken {
  identifier String
  token      String
  expires    DateTime
 
  @@id([identifier, token])
}
```
### 5.7
Dans la console:
npm exec prisma migrate dev

### 5.8
Modifier le fichier eslint.config.mjs de la manière suivante

```
const eslintConfig = [
  ...compat.extends("next/core-web-vitals", "next/typescript"),
  globalIgnores(["app/generated/prisma/"])
];
```