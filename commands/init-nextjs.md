---
description: Initialize a Next.js 15 App Router project with shadcn/ui, Zustand, and Tanstack Query following AntStack best practices
allowed-tools: [Bash, Write, Read, TodoWrite, AskUserQuestion, ReadMcpResourceTool, ListMcpResourcesTool]
---

Initialize a Next.js 15 project with App Router, Turbopack, shadcn/ui, Zustand, and Tanstack Query in the current directory.

## Before Starting

1. **Create TodoWrite list** to track initialization progress:
   - Break down the setup into measurable subtasks
   - Mark each subtask as completed when done
   - Track any issues encountered

2. **Gather required information** using AskUserQuestion:
   - Project name (if creating in a new directory)
   - TypeScript strict mode preference (recommended: yes)
   - Initial shadcn/ui components to install (e.g., button, card, input, form)
   - Application description/purpose

## Prerequisites Check

Before proceeding, verify:
- Node.js 18+ installed: `node --version`
- npm 9+ installed: `npm --version`
- Git initialized (if needed): `git --version`

## Core Requirements

- **Framework**: Next.js 15 with App Router and Turbopack
- **Directory Structure**: Use `src/` directory for source code
- **Styling**: shadcn/ui (latest) with Tailwind CSS
- **State Management**: Zustand with TypeScript
- **Data Fetching**: Tanstack Query (React Query v5)
- **Type Safety**: TypeScript strict mode

---

## Documentation Requirements

**Use Context7 MCP** for current documentation. Query before each setup phase:

1. Before Next.js setup: "Next.js 15 App Router installation with TypeScript and Turbopack"
2. Before Tailwind: "Tailwind CSS latest configuration for Next.js 15"
3. Before shadcn/ui: "shadcn/ui installation and component configuration"
4. Before Zustand: "Zustand store setup with TypeScript"
5. Before Tanstack Query: "Tanstack React Query v5 setup with App Router"

---

## Step 1: Initialize Next.js Project

Execute:

```bash
npx create-next-app@latest . --typescript --tailwind --eslint --app --src-dir --no-git --use-npm
```

**Note**: Using `.` installs in the current directory. Adjust if creating a new subdirectory.

**Verify**:
- `src/app/` directory exists
- `tailwind.config.ts` exists
- `package.json` created with Next.js 15
- `tsconfig.json` configured

**If strict TypeScript is desired**, update `tsconfig.json`:

```json
{
  "compilerOptions": {
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true
  }
}
```

---

## Step 2: Configure Development Scripts

Update `package.json` scripts to use Turbopack:

```json
{
  "scripts": {
    "dev": "next dev --turbo",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "type-check": "tsc --noEmit"
  }
}
```

---

## Step 3: Initialize shadcn/ui

Execute:

```bash
npx shadcn@latest init -d
```

**Verify**:
- `components.json` exists in root
- `src/components/ui/` directory created
- `src/lib/utils.ts` created with `cn` utility

---

## Step 4: Install Essential shadcn/ui Components

Install commonly used components:

```bash
npx shadcn@latest add button card input label form dialog toast
```

**Verify**: Check `src/components/ui/` contains installed components

---

## Step 5: Install Zustand

```bash
npm install zustand
```

Create example store at `src/stores/example-store.ts`:

```typescript
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';

interface User {
  id: string;
  name: string;
  email: string;
}

interface UserState {
  user: User | null;
  isAuthenticated: boolean;
  setUser: (user: User) => void;
  clearUser: () => void;
}

export const useUserStore = create<UserState>()(
  devtools(
    persist(
      (set) => ({
        user: null,
        isAuthenticated: false,
        setUser: (user) =>
          set({ user, isAuthenticated: true }, false, 'setUser'),
        clearUser: () =>
          set({ user: null, isAuthenticated: false }, false, 'clearUser'),
      }),
      {
        name: 'user-storage',
      }
    )
  )
);
```

Create `src/stores/index.ts` to export all stores:

```typescript
export { useUserStore } from './example-store';
// Export other stores here as you create them
```

**Verify**: Store file created at `src/stores/example-store.ts`

---

## Step 6: Install and Configure Tanstack Query

```bash
npm install @tanstack/react-query @tanstack/react-query-devtools
```

Create query provider at `src/providers/query-provider.tsx`:

```typescript
'use client';

import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
import { useState } from 'react';

export function QueryProvider({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(
    () =>
      new QueryClient({
        defaultOptions: {
          queries: {
            staleTime: 60 * 1000, // 1 minute
            refetchOnWindowFocus: false,
          },
        },
      })
  );

  return (
    <QueryClientProvider client={queryClient}>
      {children}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}
```

Update `src/app/layout.tsx` to include the provider:

```typescript
import type { Metadata } from 'next';
import { Inter } from 'next/font/google';
import './globals.css';
import { QueryProvider } from '@/providers/query-provider';

const inter = Inter({ subsets: ['latin'] });

export const metadata: Metadata = {
  title: 'Create Next App',
  description: 'Generated by create next app',
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body className={inter.className}>
        <QueryProvider>{children}</QueryProvider>
      </body>
    </html>
  );
}
```

**Verify**: Provider created and integrated in root layout

---

## Step 7: Create Example API Hook

Create `src/hooks/use-example-query.ts`:

```typescript
import { useQuery } from '@tanstack/react-query';

interface ExampleData {
  id: string;
  title: string;
  description: string;
}

async function fetchExample(): Promise<ExampleData> {
  const response = await fetch('https://api.example.com/data');
  if (!response.ok) {
    throw new Error('Network response was not ok');
  }
  return response.json();
}

export function useExampleQuery() {
  return useQuery({
    queryKey: ['example'],
    queryFn: fetchExample,
  });
}
```

**Verify**: Hook file created at `src/hooks/use-example-query.ts`

---

## Step 8: Create Example Component Integration

Create `src/components/example-component.tsx` demonstrating all integrations:

```typescript
'use client';

import { Button } from '@/components/ui/button';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import { useUserStore } from '@/stores';
import { useExampleQuery } from '@/hooks/use-example-query';

export function ExampleComponent() {
  const { user, setUser, clearUser } = useUserStore();
  const { data, isLoading, error } = useExampleQuery();

  const handleLogin = () => {
    setUser({
      id: '1',
      name: 'John Doe',
      email: 'john@example.com',
    });
  };

  return (
    <Card className="w-full max-w-md">
      <CardHeader>
        <CardTitle>Example Component</CardTitle>
        <CardDescription>
          Demonstrates shadcn/ui, Zustand, and Tanstack Query
        </CardDescription>
      </CardHeader>
      <CardContent className="space-y-4">
        {/* Zustand State Management */}
        <div>
          <p className="text-sm font-medium">User Status:</p>
          {user ? (
            <div>
              <p className="text-sm text-muted-foreground">
                Logged in as {user.name}
              </p>
              <Button onClick={clearUser} variant="outline" size="sm">
                Logout
              </Button>
            </div>
          ) : (
            <Button onClick={handleLogin} size="sm">
              Login
            </Button>
          )}
        </div>

        {/* Tanstack Query Data Fetching */}
        <div>
          <p className="text-sm font-medium">API Data:</p>
          {isLoading && <p className="text-sm">Loading...</p>}
          {error && <p className="text-sm text-red-500">Error loading data</p>}
          {data && (
            <p className="text-sm text-muted-foreground">{data.title}</p>
          )}
        </div>
      </CardContent>
    </Card>
  );
}
```

**Verify**: Component file created with all integrations

---

## Step 9: Update Home Page

Update `src/app/page.tsx`:

```typescript
import { ExampleComponent } from '@/components/example-component';

export default function Home() {
  return (
    <main className="flex min-h-screen flex-col items-center justify-center p-24">
      <div className="z-10 max-w-5xl w-full items-center justify-center font-mono text-sm">
        <h1 className="text-4xl font-bold mb-8 text-center">
          Next.js 15 + shadcn/ui + Zustand + Tanstack Query
        </h1>
        <ExampleComponent />
      </div>
    </main>
  );
}
```

---

## Step 10: Create .gitignore (if not exists)

Ensure `.gitignore` includes:

```
# Dependencies
node_modules/
.pnp
.pnp.js

# Next.js
.next/
out/
build/
dist/

# Environment
.env
.env.local
.env.development.local
.env.test.local
.env.production.local

# Testing
coverage/
.nyc_output/

# Misc
.DS_Store
*.pem

# Debug
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# IDE
.vscode/
.idea/
*.swp
*.swo

# Turbopack
.turbo/
```

---

## Step 11: Final Verification

Run these commands to verify everything works:

```bash
# Type checking
npm run type-check

# Linting
npm run lint

# Build for production
npm run build

# Start development server
npm run dev
```

**Verify**:
- No TypeScript errors
- No linting errors
- Production build succeeds
- Dev server starts successfully
- Visit http://localhost:3000 to see the example component

---

## Troubleshooting

**If `create-next-app` fails**:
- Ensure Node.js 18+ is installed: `node --version`
- Clear npm cache: `npm cache clean --force`
- Try with explicit version: `npx create-next-app@15 .`

**If shadcn/ui init fails**:
- Ensure Tailwind CSS is properly configured
- Manually create `components.json` if needed
- Check that `src/` directory structure exists

**If Turbopack fails to start**:
- Remove `--turbo` flag from dev script temporarily
- Check for conflicting packages
- Update Next.js: `npm install next@latest`

**If TypeScript errors occur**:
- Run: `npm install @types/node @types/react @types/react-dom`
- Check `tsconfig.json` paths configuration
- Ensure all imports use correct path aliases (@/)

**If Zustand persist errors occur**:
- Wrap Zustand usage in components with 'use client' directive
- Check browser localStorage is available
- Add error boundaries around Zustand components

**If Tanstack Query errors occur**:
- Ensure QueryProvider wraps entire app
- Check 'use client' directive on components using hooks
- Verify query keys are unique and consistent

---

## Success Criteria

✅ Next.js 15 running on `npm run dev`
✅ No TypeScript compilation errors
✅ No ESLint errors
✅ Tailwind CSS classes render correctly
✅ At least 5 shadcn/ui components installed in `src/components/ui/`
✅ Zustand store created with TypeScript types and devtools
✅ QueryClientProvider configured in root layout
✅ Example component demonstrates all integrations
✅ Production build completes successfully (`npm run build`)
✅ Development server uses Turbopack

---

## Next Steps

After successful initialization:
1. Configure additional shadcn/ui themes
2. Add more Zustand stores for different app states
3. Create API route handlers in `src/app/api/`
4. Set up authentication with NextAuth.js
5. Add form validation with React Hook Form + Zod
6. Configure environment variables
7. Set up testing with Jest and React Testing Library
8. Add error boundaries and loading states
9. Configure middleware for authentication/authorization
10. Optimize images and fonts

---

## Project Structure

Your project should now have:

```
.
├── src/
│   ├── app/                     # Next.js App Router
│   │   ├── layout.tsx          # Root layout with providers
│   │   ├── page.tsx            # Home page
│   │   ├── globals.css         # Global styles
│   │   └── api/                # API routes (optional)
│   ├── components/
│   │   ├── ui/                 # shadcn/ui components
│   │   │   ├── button.tsx
│   │   │   ├── card.tsx
│   │   │   └── ...
│   │   └── example-component.tsx
│   ├── hooks/
│   │   └── use-example-query.ts
│   ├── lib/
│   │   └── utils.ts            # Utility functions (cn, etc.)
│   ├── providers/
│   │   └── query-provider.tsx  # Tanstack Query provider
│   └── stores/
│       ├── index.ts            # Store exports
│       └── example-store.ts    # Zustand store
├── public/                      # Static assets
├── node_modules/
├── .gitignore
├── components.json              # shadcn/ui config
├── next.config.js               # Next.js configuration
├── package.json
├── postcss.config.js            # PostCSS config
├── tailwind.config.ts           # Tailwind configuration
└── tsconfig.json                # TypeScript configuration
```

---

## AntStack Frontend Standards Compliance

This setup follows AntStack frontend standards:
- ✅ Component structure with proper organization
- ✅ TypeScript strict mode for type safety
- ✅ State management with Zustand
- ✅ Data fetching with Tanstack Query
- ✅ UI consistency with shadcn/ui
- ✅ Tailwind CSS for styling
- ✅ Next.js App Router for routing
- ✅ Proper folder structure (src/ directory)
