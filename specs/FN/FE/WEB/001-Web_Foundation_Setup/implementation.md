# Implementation - FN/FE/WEB/001

## 1. Tech Stack

- **Framework**: Next.js 15.x (App Router)
- **Language**: TypeScript 5.x (strict mode)
- **Database**: PostgreSQL 16.x (Docker container)
- **ORM**: Prisma 5.x
- **UI**: shadcn/ui + Tailwind CSS 3.x
- **Package Manager**: npm (or pnpm)
- **Node Version**: v22.11.0 (via nvm)

---

## 2. Project Structure

```
/web
├── .nvmrc                      # v22.11.0
├── .env.local                  # Local environment variables
├── .gitignore
├── package.json
├── tsconfig.json               # strict: true
├── next.config.js
├── tailwind.config.ts
├── components.json             # shadcn/ui config
├── postcss.config.js
│
├── /prisma
│   ├── schema.prisma
│   └── /migrations
│
├── /src
│   ├── /app
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   ├── /api
│   │   │   └── /users
│   │   │       └── route.ts    # Example API route
│   │   └── /test-db
│   │       └── page.tsx        # Test DB connection
│   │
│   ├── /components
│   │   └── /ui                 # shadcn/ui components
│   │
│   ├── /lib
│   │   ├── utils.ts            # cn() helper
│   │   └── prisma.ts           # Prisma client singleton
│   │
│   └── /types
│       └── index.ts
│
└── /public
```

---

## 3. Configuration Files

### 3.1 `.nvmrc`

```
v22.11.0
```

### 3.2 `.env.local`

```bash
# Database
DATABASE_URL="postgresql://postgres:dev_password@localhost:5432/orto_scheduler?schema=public"

# App Config
NEXT_PUBLIC_APP_NAME="Universal Scheduler"
```

### 3.3 `prisma/schema.prisma`

```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

// Basic User model for testing
// Will expand with Staff, Constraints, Schedules later
model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String
  role      String   @default("user") // user, admin
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([email])
}
```

### 3.4 `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [
      {
        "name": "next"
      }
    ],
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

### 3.5 `tailwind.config.ts`

```typescript
import type { Config } from "tailwindcss"

const config = {
  darkMode: ["class"],
  content: [
    './pages/**/*.{ts,tsx}',
    './components/**/*.{ts,tsx}',
    './app/**/*.{ts,tsx}',
    './src/**/*.{ts,tsx}',
  ],
  prefix: "",
  theme: {
    container: {
      center: true,
      padding: "2rem",
      screens: {
        "2xl": "1400px",
      },
    },
    extend: {
      colors: {
        border: "hsl(var(--border))",
        input: "hsl(var(--input))",
        ring: "hsl(var(--ring))",
        background: "hsl(var(--background))",
        foreground: "hsl(var(--foreground))",
        primary: {
          DEFAULT: "hsl(var(--primary))",
          foreground: "hsl(var(--primary-foreground))",
        },
        secondary: {
          DEFAULT: "hsl(var(--secondary))",
          foreground: "hsl(var(--secondary-foreground))",
        },
        destructive: {
          DEFAULT: "hsl(var(--destructive))",
          foreground: "hsl(var(--destructive-foreground))",
        },
        muted: {
          DEFAULT: "hsl(var(--muted))",
          foreground: "hsl(var(--muted-foreground))",
        },
        accent: {
          DEFAULT: "hsl(var(--accent))",
          foreground: "hsl(var(--accent-foreground))",
        },
        popover: {
          DEFAULT: "hsl(var(--popover))",
          foreground: "hsl(var(--popover-foreground))",
        },
        card: {
          DEFAULT: "hsl(var(--card))",
          foreground: "hsl(var(--card-foreground))",
        },
      },
      borderRadius: {
        lg: "var(--radius)",
        md: "calc(var(--radius) - 2px)",
        sm: "calc(var(--radius) - 4px)",
      },
      keyframes: {
        "accordion-down": {
          from: { height: "0" },
          to: { height: "var(--radix-accordion-content-height)" },
        },
        "accordion-up": {
          from: { height: "var(--radix-accordion-content-height)" },
          to: { height: "0" },
        },
      },
      animation: {
        "accordion-down": "accordion-down 0.2s ease-out",
        "accordion-up": "accordion-up 0.2s ease-out",
      },
    },
  },
  plugins: [require("tailwindcss-animate")],
} satisfies Config

export default config
```

---

## 4. Key Implementation Files

### 4.1 `/src/lib/prisma.ts` - Prisma Client Singleton

```typescript
import { PrismaClient } from '@prisma/client'

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined
}

export const prisma = globalForPrisma.prisma ?? new PrismaClient({
  log: process.env.NODE_ENV === 'development' ? ['query', 'error', 'warn'] : ['error'],
})

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma
```

### 4.2 `/src/lib/utils.ts` - Tailwind Helper

```typescript
import { type ClassValue, clsx } from "clsx"
import { twMerge } from "tailwind-merge"

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

### 4.3 `/src/app/test-db/page.tsx` - Database Test Page

```typescript
import { prisma } from '@/lib/prisma'

export default async function TestDbPage() {
  // Fetch users from database
  const users = await prisma.user.findMany({
    orderBy: { createdAt: 'desc' },
    take: 10,
  })

  const userCount = await prisma.user.count()

  return (
    <div className="container mx-auto p-8">
      <h1 className="text-3xl font-bold mb-4">Database Connection Test</h1>
      
      <div className="bg-green-50 border border-green-200 rounded-lg p-4 mb-6">
        <p className="text-green-800">
          ✅ Connected to PostgreSQL successfully!
        </p>
        <p className="text-sm text-green-600 mt-2">
          Total users in database: <strong>{userCount}</strong>
        </p>
      </div>

      <div className="bg-white rounded-lg shadow p-6">
        <h2 className="text-xl font-semibold mb-4">Recent Users</h2>
        {users.length === 0 ? (
          <p className="text-gray-500">No users found. Add some via Prisma Studio!</p>
        ) : (
          <div className="space-y-2">
            {users.map((user) => (
              <div key={user.id} className="border-b pb-2">
                <p className="font-medium">{user.name}</p>
                <p className="text-sm text-gray-600">{user.email}</p>
                <p className="text-xs text-gray-400">
                  Role: {user.role} | Created: {user.createdAt.toLocaleDateString()}
                </p>
              </div>
            ))}
          </div>
        )}
      </div>

      <div className="mt-6 text-sm text-gray-500">
        <p>💡 Tip: Open Prisma Studio to manage data:</p>
        <code className="bg-gray-100 px-2 py-1 rounded">npx prisma studio</code>
      </div>
    </div>
  )
}
```

### 4.4 `/src/app/api/users/route.ts` - Example API Route

```typescript
import { NextRequest, NextResponse } from 'next/server'
import { prisma } from '@/lib/prisma'

// GET /api/users - List all users
export async function GET() {
  try {
    const users = await prisma.user.findMany({
      orderBy: { createdAt: 'desc' },
    })
    return NextResponse.json(users)
  } catch (error) {
    console.error('Error fetching users:', error)
    return NextResponse.json(
      { error: 'Failed to fetch users' },
      { status: 500 }
    )
  }
}

// POST /api/users - Create a new user
export async function POST(request: NextRequest) {
  try {
    const body = await request.json()
    const { email, name, role = 'user' } = body

    if (!email || !name) {
      return NextResponse.json(
        { error: 'Email and name are required' },
        { status: 400 }
      )
    }

    const user = await prisma.user.create({
      data: { email, name, role },
    })

    return NextResponse.json(user, { status: 201 })
  } catch (error) {
    console.error('Error creating user:', error)
    return NextResponse.json(
      { error: 'Failed to create user' },
      { status: 500 }
    )
  }
}
```

### 4.5 `/src/app/page.tsx` - Homepage

```typescript
import Link from 'next/link'
import { Button } from '@/components/ui/button'

export default function Home() {
  return (
    <main className="container mx-auto p-8">
      <div className="max-w-2xl mx-auto space-y-8">
        <div className="text-center">
          <h1 className="text-4xl font-bold mb-4">
            Universal Scheduler
          </h1>
          <p className="text-lg text-gray-600">
            Nurse Edition - Roster Management System
          </p>
        </div>

        <div className="bg-white rounded-lg shadow-lg p-8 space-y-6">
          <h2 className="text-2xl font-semibold">Getting Started</h2>
          
          <div className="space-y-4">
            <div>
              <h3 className="font-medium mb-2">✅ Foundation Setup Complete</h3>
              <ul className="list-disc list-inside text-sm text-gray-600 space-y-1">
                <li>Next.js 15 with TypeScript</li>
                <li>PostgreSQL + Prisma ORM</li>
                <li>Tailwind CSS + shadcn/ui</li>
              </ul>
            </div>

            <div className="pt-4 border-t">
              <Link href="/test-db">
                <Button>
                  Test Database Connection
                </Button>
              </Link>
            </div>
          </div>
        </div>

        <div className="text-center text-sm text-gray-500">
          <p>Next Steps: Start building features! 🚀</p>
        </div>
      </div>
    </main>
  )
}
```

---

## 5. Docker Setup for PostgreSQL

### Docker Command

```bash
docker run --name orto-postgres \
  -e POSTGRES_PASSWORD=dev_password \
  -e POSTGRES_DB=orto_scheduler \
  -p 5432:5432 \
  -d postgres:16
```

### Verify Connection

```bash
docker exec -it orto-postgres psql -U postgres -d orto_scheduler
```

### Useful Docker Commands

```bash
# Stop container
docker stop orto-postgres

# Start container
docker start orto-postgres

# Remove container (data will be lost)
docker rm orto-postgres

# View logs
docker logs orto-postgres
```

---

## 6. Installation Steps

See `task.md` for detailed step-by-step commands.

---

## 7. Expected Package Versions

```json
{
  "name": "web",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "next": "^15.0.3",
    "react": "^18.3.1",
    "react-dom": "^18.3.1",
    "@prisma/client": "^5.7.1",
    "@radix-ui/react-slot": "^1.0.2",
    "class-variance-authority": "^0.7.0",
    "clsx": "^2.1.0",
    "tailwind-merge": "^2.2.0",
    "lucide-react": "^0.294.0",
    "zod": "^3.22.4"
  },
  "devDependencies": {
    "typescript": "^5.3.3",
    "prisma": "^5.7.1",
    "@types/node": "^20.10.6",
    "@types/react": "^18.2.46",
    "@types/react-dom": "^18.2.18",
    "tailwindcss": "^3.4.0",
    "tailwindcss-animate": "^1.0.7",
    "postcss": "^8.4.32",
    "autoprefixer": "^10.4.16",
    "eslint": "^8.56.0",
    "eslint-config-next": "^15.0.3"
  }
}
```

---

## 8. Verification Checklist

After setup, verify:

- [ ] `npm run dev` starts Next.js on `http://localhost:3000`
- [ ] No TypeScript errors (`npm run build` succeeds)
- [ ] PostgreSQL container is running (`docker ps | grep orto-postgres`)
- [ ] Prisma can connect to DB (`npx prisma studio` opens)
- [ ] `/test-db` page loads without errors
- [ ] shadcn/ui Button component renders correctly
- [ ] Can create a new user via API: `POST /api/users`
- [ ] Can view users in Prisma Studio and see count update on `/test-db`
- [ ] Home page navigation works without console errors
