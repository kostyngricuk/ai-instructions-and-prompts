---
description: 'Next.js + Tailwind development standards and instructions'
applyTo: '**/*.tsx, **/*.ts, **/*.jsx, **/*.js, **/*.css'
---

# Next.js + Tailwind + TypeScript Development Instructions

Comprehensive instructions for AI tools to generate high-quality, production-ready Next.js applications using the most popular modern stack.

## Modern Tech Stack

### Core Framework
- **Next.js 15+** with App Router (stable)
- **TypeScript 5+** with strict mode
- **React 18+** with Server Components
- **Tailwind CSS 3+** for styling

### Essential Dependencies
- **@next/font** - Font optimization
- **next/image** - Image optimization
- **zod** - Runtime type validation
- **clsx** or **cn** utility - Conditional classes
- **tailwind-merge** - Merge Tailwind classes
- **@mui/material** - Material-UI base components
- **@mui/icons-material** - Material-UI icons
- **@emotion/react** & **@emotion/styled** - MUI styling dependencies

### Development Tools
- **ESLint** with Next.js config
- **Prettier** with Tailwind plugin
- **TypeScript ESLint** rules
- **Husky** + **lint-staged** for git hooks

## Project Structure

```
src/
├── app/                          # App Router pages
│   ├── (auth)/                   # Route groups
│   ├── api/                      # API routes
│   ├── globals.css               # Global styles
│   ├── layout.tsx                # Root layout
│   └── page.tsx                  # Home page
├── components/                   # Reusable components
│   ├── ui/                       # Base UI components
│   ├── forms/                    # Form components
│   └── layout/                   # Layout components
├── lib/                          # Utilities and config
│   ├── utils.ts                  # Utility functions
│   ├── validations.ts            # Zod schemas
│   └── constants.ts              # App constants
├── hooks/                        # Custom React hooks
├── types/                        # TypeScript type definitions
└── styles/                       # Additional stylesheets
```

## Core Development Principles

### 1. TypeScript Best Practices
- Use strict mode: `"strict": true` in tsconfig.json
- Define explicit interfaces for all props and API responses
- Use type assertions sparingly, prefer type guards
- Implement proper error types with discriminated unions
- Use `satisfies` operator for better type inference

```typescript
// Good: Explicit interface
interface UserProps {
  user: {
    id: string;
    name: string;
    email: string;
  };
  onUpdate: (user: User) => void;
}

// Good: Type guard
function isError(value: unknown): value is Error {
  return value instanceof Error;
}
```

### 2. Component Architecture
- **Server Components by default** - Use for data fetching and static content
- **Client Components** - Only when needed for interactivity
- **Composition over inheritance** - Build complex UIs from simple components
- **Single Responsibility Principle** - One component, one purpose

```typescript
// Server Component (default)
async function UserProfile({ userId }: { userId: string }) {
  const user = await getUser(userId);
  return <UserCard user={user} />;
}

// Client Component (when needed)
'use client';
function InteractiveButton({ onClick }: { onClick: () => void }) {
  const [isLoading, setIsLoading] = useState(false);
  // ... interactive logic
}
```

### 3. Styling with MUI + Tailwind CSS

#### MUI Component Integration
- Use MUI components as base building blocks
- Enhance with Tailwind utilities for spacing, layout, and custom styling
- Leverage MUI's theming system for consistent design
- Use Tailwind for responsive design and utility classes

```typescript
import { Button as MuiButton, ButtonProps } from '@mui/material';
import { cn } from '@/lib/utils';

interface CustomButtonProps extends ButtonProps {
  customVariant?: 'gradient' | 'outlined-custom';
}

function Button({ customVariant, className, ...props }: CustomButtonProps) {
  const customClasses = {
    gradient: 'bg-gradient-to-r from-blue-500 to-purple-600 text-white',
    'outlined-custom': 'border-2 border-blue-500 text-blue-500 hover:bg-blue-50',
  };

  if (customVariant) {
    return (
      <button
        className={cn(
          'px-4 py-2 rounded-md font-medium transition-colors',
          customClasses[customVariant],
          className
        )}
        {...props}
      />
    );
  }

  return <MuiButton className={cn('normal-case', className)} {...props} />;
}
```

#### MUI Theme Integration
```typescript
// app/theme.ts
import { createTheme } from '@mui/material/styles';

export const theme = createTheme({
  palette: {
    primary: {
      main: '#3b82f6', // blue-500
    },
    secondary: {
      main: '#8b5cf6', // violet-500
    },
  },
  components: {
    MuiButton: {
      styleOverrides: {
        root: {
          textTransform: 'none',
          borderRadius: '0.375rem', // rounded-md
        },
      },
    },
  },
});
```

#### Responsive Design Patterns
```typescript
// Mobile-first responsive design
<div className="
  grid grid-cols-1 gap-4
  sm:grid-cols-2 sm:gap-6
  lg:grid-cols-3 lg:gap-8
  xl:grid-cols-4
">
```

### 4. Data Fetching Strategy

#### Server Components for Data
```typescript
// Fetch data directly in Server Components
async function ProductList({ category }: { category: string }) {
  const products = await fetch(`/api/products?category=${category}`, {
    cache: 'force-cache', // or 'no-store' for dynamic data
  }).then(res => res.json());

  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}
```

#### Client-Side Data with React Query (Optional)
```typescript
'use client';
import { useQuery } from '@tanstack/react-query';

function useUser(userId: string) {
  return useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
    staleTime: 5 * 60 * 1000, // 5 minutes
  });
}
```

### 5. Form Handling with MUI Components
- Use **React Hook Form** with **Zod** validation
- Integrate MUI form components with React Hook Form
- Implement proper error states and loading indicators
- Use Server Actions for form submissions when possible

```typescript
import { useForm, Controller } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { TextField, Button, Box, Alert } from '@mui/material';

const userSchema = z.object({
  name: z.string().min(2, 'Name must be at least 2 characters'),
  email: z.string().email('Invalid email address'),
});

type UserFormData = z.infer<typeof userSchema>;

function UserForm() {
  const { control, handleSubmit, formState: { errors, isSubmitting } } = useForm<UserFormData>({
    resolver: zodResolver(userSchema),
    defaultValues: {
      name: '',
      email: '',
    },
  });

  const onSubmit = async (data: UserFormData) => {
    // Handle form submission
  };

  return (
    <Box component="form" onSubmit={handleSubmit(onSubmit)} className="space-y-4 max-w-md mx-auto">
      <Controller
        name="name"
        control={control}
        render={({ field }) => (
          <TextField
            {...field}
            label="Name"
            fullWidth
            error={!!errors.name}
            helperText={errors.name?.message}
            className="mb-4"
          />
        )}
      />
      
      <Controller
        name="email"
        control={control}
        render={({ field }) => (
          <TextField
            {...field}
            label="Email"
            type="email"
            fullWidth
            error={!!errors.email}
            helperText={errors.email?.message}
            className="mb-4"
          />
        )}
      />
      
      <Button
        type="submit"
        variant="contained"
        fullWidth
        disabled={isSubmitting}
        className="mt-4"
      >
        {isSubmitting ? 'Submitting...' : 'Submit'}
      </Button>
    </Box>
  );
}
```

### 6. API Routes and Server Actions

#### API Routes (app/api)
```typescript
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { z } from 'zod';

const createUserSchema = z.object({
  name: z.string().min(2),
  email: z.string().email(),
});

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { name, email } = createUserSchema.parse(body);
    
    // Create user logic
    const user = await createUser({ name, email });
    
    return NextResponse.json(user, { status: 201 });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json({ error: error.errors }, { status: 400 });
    }
    return NextResponse.json({ error: 'Internal server error' }, { status: 500 });
  }
}
```

#### Server Actions (Preferred for Forms)
```typescript
// app/actions/users.ts
'use server';
import { z } from 'zod';
import { revalidatePath } from 'next/cache';

const userSchema = z.object({
  name: z.string().min(2),
  email: z.string().email(),
});

export async function createUser(formData: FormData) {
  const result = userSchema.safeParse({
    name: formData.get('name'),
    email: formData.get('email'),
  });

  if (!result.success) {
    return { error: result.error.flatten() };
  }

  // Create user in database
  const user = await db.user.create({ data: result.data });
  
  revalidatePath('/users');
  return { success: true, user };
}
```

### 7. Error Handling and Loading States

#### Error Boundaries
```typescript
// app/error.tsx
'use client';
import { useEffect } from 'react';

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    console.error(error);
  }, [error]);

  return (
    <div className="flex min-h-screen flex-col items-center justify-center">
      <div className="text-center">
        <h2 className="text-2xl font-bold text-gray-900 mb-4">Something went wrong!</h2>
        <button
          onClick={reset}
          className="px-4 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700"
        >
          Try again
        </button>
      </div>
    </div>
  );
}
```

#### Loading UI
```typescript
// app/loading.tsx
export default function Loading() {
  return (
    <div className="flex items-center justify-center min-h-screen">
      <div className="animate-spin rounded-full h-32 w-32 border-b-2 border-blue-600"></div>
    </div>
  );
}
```

### 8. Performance Optimization

#### Image Optimization
```typescript
import Image from 'next/image';

function ProductImage({ product }: { product: Product }) {
  return (
    <Image
      src={product.image}
      alt={product.name}
      width={400}
      height={300}
      className="rounded-lg object-cover"
      placeholder="blur"
      blurDataURL="data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQ..."
      sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
      priority={product.featured}
    />
  );
}
```

#### Font Optimization with MUI Theme
```typescript
// app/layout.tsx
import { Inter, Roboto_Mono } from 'next/font/google';
import { ThemeProvider } from '@mui/material/styles';
import { CssBaseline } from '@mui/material';
import { theme } from './theme';

const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-inter',
});

const robotoMono = Roboto_Mono({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-roboto-mono',
});

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" className={`${inter.variable} ${robotoMono.variable}`}>
      <body className="font-sans antialiased">
        <ThemeProvider theme={theme}>
          <CssBaseline />
          {children}
        </ThemeProvider>
      </body>
    </html>
  );
}
```

### 9. Accessibility Best Practices

#### Accessibility with MUI Components
```typescript
import { 
  TextField, 
  Button, 
  FormControl, 
  InputLabel, 
  Select, 
  MenuItem,
  FormHelperText,
  Box 
} from '@mui/material';
import { Search as SearchIcon } from '@mui/icons-material';

function SearchForm() {
  const [searchQuery, setSearchQuery] = useState('');
  const [category, setCategory] = useState('');

  return (
    <Box component="form" role="search" className="flex gap-4 items-start">
      <TextField
        id="search"
        label="Search products"
        value={searchQuery}
        onChange={(e) => setSearchQuery(e.target.value)}
        placeholder="Enter keywords..."
        className="flex-1"
        InputProps={{
          startAdornment: <SearchIcon className="text-gray-400 mr-2" />,
        }}
        helperText="Enter keywords to search for products"
      />
      
      <FormControl className="min-w-32">
        <InputLabel id="category-label">Category</InputLabel>
        <Select
          labelId="category-label"
          value={category}
          label="Category"
          onChange={(e) => setCategory(e.target.value)}
        >
          <MenuItem value="">All</MenuItem>
          <MenuItem value="electronics">Electronics</MenuItem>
          <MenuItem value="clothing">Clothing</MenuItem>
        </Select>
      </FormControl>
      
      <Button
        type="submit"
        variant="contained"
        aria-label="Submit search"
        className="h-14"
      >
        Search
      </Button>
    </Box>
  );
}
```

## Naming Conventions and File Organization

### File Naming
- **Components**: `PascalCase.tsx` (e.g., `UserCard.tsx`)
- **Pages**: `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`
- **Hooks**: `use*.ts` (e.g., `useUser.ts`)
- **Utilities**: `camelCase.ts` (e.g., `formatDate.ts`)
- **Types**: `types.ts` or `*.types.ts`
- **Constants**: `constants.ts` or `UPPER_SNAKE_CASE`

### Component Organization with MUI
```typescript
// components/ui/Button/index.ts
export { Button } from './Button';
export type { ButtonProps } from './Button';

// components/ui/Button/Button.tsx
import { forwardRef } from 'react';
import { Button as MuiButton, ButtonProps as MuiButtonProps } from '@mui/material';
import { cn } from '@/lib/utils';

export interface ButtonProps extends MuiButtonProps {
  customVariant?: 'gradient' | 'outlined-custom';
}

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, customVariant, children, ...props }, ref) => {
    const customClasses = {
      gradient: 'bg-gradient-to-r from-blue-500 to-purple-600 text-white hover:from-blue-600 hover:to-purple-700',
      'outlined-custom': 'border-2 border-blue-500 text-blue-500 hover:bg-blue-50',
    };

    if (customVariant) {
      return (
        <button
          className={cn(
            'px-4 py-2 rounded-md font-medium transition-colors',
            customClasses[customVariant],
            className
          )}
          ref={ref}
          {...props}
        >
          {children}
        </button>
      );
    }

    return (
      <MuiButton
        className={cn('normal-case', className)}
        ref={ref}
        {...props}
      >
        {children}
      </MuiButton>
    );
  }
);

Button.displayName = 'Button';
```

## Security Best Practices

### Input Validation
```typescript
// Always validate input on both client and server
const userSchema = z.object({
  email: z.string().email().max(255),
  password: z.string().min(8).max(100),
  name: z.string().min(2).max(50).regex(/^[a-zA-Z\s]+$/),
});

// Sanitize HTML content
import DOMPurify from 'isomorphic-dompurify';

function sanitizeHTML(html: string) {
  return DOMPurify.sanitize(html);
}
```

### Environment Variables
```typescript
// lib/env.ts
import { z } from 'zod';

const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  NEXTAUTH_SECRET: z.string().min(32),
  NEXTAUTH_URL: z.string().url(),
});

export const env = envSchema.parse(process.env);
```

## Code Quality Tools Configuration

### ESLint Configuration
```json
// .eslintrc.json
{
  "extends": [
    "next/core-web-vitals",
    "@typescript-eslint/recommended",
    "prettier"
  ],
  "rules": {
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/no-explicit-any": "error",
    "prefer-const": "error"
  }
}
```

### Prettier Configuration
```json
// .prettierrc
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "tabWidth": 2,
  "plugins": ["prettier-plugin-tailwindcss"]
}
```

## Production Deployment Checklist

### Performance
- [ ] Images optimized with next/image
- [ ] Fonts optimized with next/font
- [ ] Bundle analyzed with @next/bundle-analyzer
- [ ] Core Web Vitals passing
- [ ] Caching strategies implemented

### Security
- [ ] Environment variables secured
- [ ] HTTPS enabled
- [ ] Security headers configured
- [ ] Input validation implemented
- [ ] Authentication/authorization working

### SEO
- [ ] Meta tags implemented
- [ ] Sitemap.xml generated
- [ ] robots.txt configured
- [ ] Structured data added
- [ ] Open Graph tags included

## Common Patterns and Examples

### Custom Hook Pattern
```typescript
// hooks/useLocalStorage.ts
import { useState, useEffect } from 'react';

export function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    if (typeof window === 'undefined') return initialValue;
    
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(`Error reading localStorage key "${key}":`, error);
      return initialValue;
    }
  });

  const setValue = (value: T | ((val: T) => T)) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      
      if (typeof window !== 'undefined') {
        window.localStorage.setItem(key, JSON.stringify(valueToStore));
      }
    } catch (error) {
      console.error(`Error setting localStorage key "${key}":`, error);
    }
  };

  return [storedValue, setValue] as const;
}
```

### Context Provider Pattern with MUI
```typescript
// contexts/ThemeContext.tsx
'use client';
import { createContext, useContext, useEffect, useState } from 'react';
import { ThemeProvider as MuiThemeProvider, createTheme } from '@mui/material/styles';
import { CssBaseline } from '@mui/material';

type Theme = 'light' | 'dark' | 'system';

interface ThemeContextType {
  theme: Theme;
  setTheme: (theme: Theme) => void;
  resolvedTheme: 'light' | 'dark';
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<Theme>('system');
  const [resolvedTheme, setResolvedTheme] = useState<'light' | 'dark'>('light');

  useEffect(() => {
    const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
    const handleChange = () => {
      if (theme === 'system') {
        setResolvedTheme(mediaQuery.matches ? 'dark' : 'light');
      }
    };

    if (theme === 'system') {
      setResolvedTheme(mediaQuery.matches ? 'dark' : 'light');
      mediaQuery.addEventListener('change', handleChange);
      return () => mediaQuery.removeEventListener('change', handleChange);
    } else {
      setResolvedTheme(theme);
    }
  }, [theme]);

  const muiTheme = createTheme({
    palette: {
      mode: resolvedTheme,
      primary: {
        main: resolvedTheme === 'dark' ? '#60a5fa' : '#3b82f6',
      },
      secondary: {
        main: resolvedTheme === 'dark' ? '#a78bfa' : '#8b5cf6',
      },
    },
  });

  return (
    <ThemeContext.Provider value={{ theme, setTheme, resolvedTheme }}>
      <MuiThemeProvider theme={muiTheme}>
        <CssBaseline />
        {children}
      </MuiThemeProvider>
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
}
```

## AI Generation Guidelines

When generating Next.js + Tailwind + TypeScript code with MUI:

1. **Always use TypeScript** with explicit types and interfaces
2. **Prefer Server Components** unless interactivity is required
3. **Use MUI components as base** enhanced with Tailwind utilities
4. **Implement MUI theming** for consistent design system
5. **Include proper error handling** with try-catch and error boundaries
6. **Include loading states** for async operations
7. **Follow responsive design** principles using both MUI breakpoints and Tailwind
8. **Add proper accessibility** with MUI's built-in ARIA support
9. **Validate all inputs** with Zod schemas and MUI form components
10. **Use modern React patterns** (hooks, context, suspense)
11. **Optimize for performance** with Next.js built-in features
12. **Wrap components in ThemeProvider** for consistent MUI theming

### Do Not Generate
- Custom CSS files (use MUI theme + Tailwind utilities)
- Class components (use functional components with MUI)
- Unstyled form elements (use MUI TextField, Select, etc.)
- Custom icon implementations (use @mui/icons-material)
- Inline styles (use MUI sx prop or Tailwind classes)
- Example/demo files unless explicitly requested

