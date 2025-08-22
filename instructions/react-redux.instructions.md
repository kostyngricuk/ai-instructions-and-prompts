---
description: 'React + Redux Toolkit development standards and instructions'
applyTo: '**/*.tsx, **/*.ts, **/*.jsx, **/*.js, **/*.css'
---

# React + Redux Toolkit + TypeScript Development Instructions

Comprehensive instructions for AI tools to generate high-quality, production-ready React applications using Redux Toolkit for state management and the most popular modern stack.

## Modern Tech Stack

### Core Framework
- **React 18+** with concurrent features
- **TypeScript 5+** with strict mode
- **Redux Toolkit (RTK)** for state management
- **React Router 6+** for routing
- **Tailwind CSS 3+** for styling

### Essential Dependencies
- **@reduxjs/toolkit** - Redux Toolkit for state management
- **react-redux** - React bindings for Redux
- **@reduxjs/toolkit/query/react** - RTK Query for data fetching
- **react-router-dom** - Declarative routing
- **zod** - Runtime type validation
- **clsx** or **cn** utility - Conditional classes
- **tailwind-merge** - Merge Tailwind classes
- **@mui/material** - Material-UI base components
- **@mui/icons-material** - Material-UI icons
- **@emotion/react** & **@emotion/styled** - MUI styling dependencies

### Development Tools
- **Vite** - Fast build tool and dev server
- **ESLint** with React config
- **Prettier** with Tailwind plugin
- **TypeScript ESLint** rules
- **Husky** + **lint-staged** for git hooks
- **@testing-library/react** - Component testing
- **@testing-library/jest-dom** - Jest DOM matchers
- **MSW (Mock Service Worker)** - API mocking for testing

## Project Structure

```
src/
├── app/                          # Redux store configuration
│   ├── store.ts                  # Store setup
│   ├── hooks.ts                  # Typed Redux hooks
│   └── middleware/               # Custom middleware
├── features/                     # Feature-based modules
│   ├── auth/                     # Authentication feature
│   │   ├── components/           # Feature-specific components
│   │   ├── hooks/                # Feature-specific hooks
│   │   ├── services/             # RTK Query API slices
│   │   ├── slice.ts              # Redux slice
│   │   └── types.ts              # Feature types
│   └── products/                 # Products feature
│       ├── components/
│       ├── hooks/
│       ├── services/
│       ├── slice.ts
│       └── types.ts
├── components/                   # Shared components
│   ├── ui/                       # Base UI components
│   ├── forms/                    # Form components
│   ├── layout/                   # Layout components
│   └── common/                   # Common reusable components
├── hooks/                        # Shared custom hooks
├── lib/                          # Utilities and config
│   ├── utils.ts                  # Utility functions
│   ├── validations.ts            # Zod schemas
│   ├── constants.ts              # App constants
│   ├── router.tsx                # Router configuration
│   ├── routerUtils.ts            # Router utilities and guards
│   └── theme.ts                  # MUI theme configuration
├── pages/                        # Route components
├── services/                     # Shared API services
├── types/                        # Shared TypeScript types
├── styles/                       # Global styles
│   └── globals.css               # Global CSS
├── App.tsx                       # Main App component
├── main.tsx                      # Entry point
└── vite-env.d.ts                 # Vite type definitions
```

## Core Development Principles

### 1. Redux Toolkit Best Practices

#### Store Configuration
```typescript
// app/store.ts
import { configureStore } from '@reduxjs/toolkit';
import { setupListeners } from '@reduxjs/toolkit/query';
import { authSlice } from '@/features/auth/slice';
import { productsApi } from '@/features/products/services/productsApi';

export const store = configureStore({
  reducer: {
    auth: authSlice.reducer,
    [productsApi.reducerPath]: productsApi.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ['persist/PERSIST', 'persist/REHYDRATE'],
      },
    }).concat(productsApi.middleware),
  devTools: process.env.NODE_ENV !== 'production',
});

setupListeners(store.dispatch);

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

#### Typed Redux Hooks
```typescript
// app/hooks.ts
import { useDispatch, useSelector, TypedUseSelectorHook } from 'react-redux';
import type { RootState, AppDispatch } from './store';

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

#### Redux Slice Pattern
```typescript
// features/auth/slice.ts
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';
import { User, LoginCredentials, AuthState } from './types';
import { authApi } from './services/authApi';

interface AuthSliceState {
  user: User | null;
  token: string | null;
  isLoading: boolean;
  error: string | null;
}

const initialState: AuthSliceState = {
  user: null,
  token: localStorage.getItem('token'),
  isLoading: false,
  error: null,
};

// Async thunk for login
export const loginUser = createAsyncThunk(
  'auth/loginUser',
  async (credentials: LoginCredentials, { rejectWithValue }) => {
    try {
      const response = await authApi.login(credentials);
      localStorage.setItem('token', response.token);
      return response;
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

export const authSlice = createSlice({
  name: 'auth',
  initialState,
  reducers: {
    logout: (state) => {
      state.user = null;
      state.token = null;
      localStorage.removeItem('token');
    },
    clearError: (state) => {
      state.error = null;
    },
    setUser: (state, action: PayloadAction<User>) => {
      state.user = action.payload;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(loginUser.pending, (state) => {
        state.isLoading = true;
        state.error = null;
      })
      .addCase(loginUser.fulfilled, (state, action) => {
        state.isLoading = false;
        state.user = action.payload.user;
        state.token = action.payload.token;
      })
      .addCase(loginUser.rejected, (state, action) => {
        state.isLoading = false;
        state.error = action.payload as string;
      });
  },
});

export const { logout, clearError, setUser } = authSlice.actions;
```

### 2. RTK Query for Data Fetching

#### API Service Definition
```typescript
// features/products/services/productsApi.ts
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';
import { Product, CreateProductRequest } from '../types';
import type { RootState } from '@/app/store';

export const productsApi = createApi({
  reducerPath: 'productsApi',
  baseQuery: fetchBaseQuery({
    baseUrl: '/api/products',
    prepareHeaders: (headers, { getState }) => {
      const token = (getState() as RootState).auth.token;
      if (token) {
        headers.set('authorization', `Bearer ${token}`);
      }
      return headers;
    },
  }),
  tagTypes: ['Product'],
  endpoints: (builder) => ({
    getProducts: builder.query<Product[], { page?: number; limit?: number }>({
      query: ({ page = 1, limit = 10 } = {}) => `?page=${page}&limit=${limit}`,
      providesTags: ['Product'],
    }),
    getProductById: builder.query<Product, string>({
      query: (id) => `/${id}`,
      providesTags: (result, error, id) => [{ type: 'Product', id }],
    }),
    createProduct: builder.mutation<Product, CreateProductRequest>({
      query: (newProduct) => ({
        url: '',
        method: 'POST',
        body: newProduct,
      }),
      invalidatesTags: ['Product'],
    }),
    updateProduct: builder.mutation<Product, { id: string; data: Partial<Product> }>({
      query: ({ id, data }) => ({
        url: `/${id}`,
        method: 'PUT',
        body: data,
      }),
      invalidatesTags: (result, error, { id }) => [{ type: 'Product', id }],
    }),
    deleteProduct: builder.mutation<void, string>({
      query: (id) => ({
        url: `/${id}`,
        method: 'DELETE',
      }),
      invalidatesTags: ['Product'],
    }),
  }),
});

export const {
  useGetProductsQuery,
  useGetProductByIdQuery,
  useCreateProductMutation,
  useUpdateProductMutation,
  useDeleteProductMutation,
} = productsApi;
```

### 3. Component Architecture with Redux

#### Connected Component Example
```typescript
// features/products/components/ProductList.tsx
import React, { useState } from 'react';
import { 
  Grid, 
  Card, 
  CardContent, 
  Typography, 
  CircularProgress, 
  Alert,
  Pagination,
  Box 
} from '@mui/material';
import { useGetProductsQuery } from '../services/productsApi';
import { ProductCard } from './ProductCard';

export const ProductList: React.FC = () => {
  const [page, setPage] = useState(1);
  const limit = 12;
  
  const {
    data: products,
    isLoading,
    isError,
    error,
    refetch,
  } = useGetProductsQuery({ page, limit });

  const handlePageChange = (event: React.ChangeEvent<unknown>, value: number) => {
    setPage(value);
  };

  if (isLoading) {
    return (
      <Box className="flex justify-center items-center min-h-64">
        <CircularProgress />
      </Box>
    );
  }

  if (isError) {
    return (
      <Alert 
        severity="error" 
        action={
          <Button color="inherit" size="small" onClick={() => refetch()}>
            Retry
          </Button>
        }
        className="mb-4"
      >
        {error?.message || 'Failed to load products'}
      </Alert>
    );
  }

  return (
    <Box className="space-y-6">
      <Grid container spacing={3}>
        {products?.map((product) => (
          <Grid item xs={12} sm={6} md={4} lg={3} key={product.id}>
            <ProductCard product={product} />
          </Grid>
        ))}
      </Grid>
      
      <Box className="flex justify-center mt-8">
        <Pagination
          count={Math.ceil((products?.total || 0) / limit)}
          page={page}
          onChange={handlePageChange}
          color="primary"
        />
      </Box>
    </Box>
  );
};
```

### 4. React Router Integration with Router Configuration

#### Router Configuration Setup
```typescript
// lib/router.tsx
import React from 'react';
import { createBrowserRouter, Navigate, Outlet } from 'react-router-dom';
import { Layout } from '@/components/layout/Layout';
import { HomePage } from '@/pages/HomePage';
import { LoginPage } from '@/pages/LoginPage';
import { ProductsPage } from '@/pages/ProductsPage';
import { ProductDetailPage } from '@/pages/ProductDetailPage';
import { ProfilePage } from '@/pages/ProfilePage';
import { NotFoundPage } from '@/pages/NotFoundPage';
import { ErrorBoundary } from '@/components/common/ErrorBoundary';
import { ProtectedRoute } from '@/components/auth/ProtectedRoute';
import { GuestRoute } from '@/components/auth/GuestRoute';

export const router = createBrowserRouter([
  {
    path: '/',
    element: <Layout />,
    errorElement: <ErrorBoundary />,
    children: [
      {
        index: true,
        element: <HomePage />,
      },
      {
        path: 'login',
        element: (
          <GuestRoute>
            <LoginPage />
          </GuestRoute>
        ),
      },
      {
        path: 'register',
        element: (
          <GuestRoute>
            <RegisterPage />
          </GuestRoute>
        ),
      },
      {
        path: 'products',
        element: (
          <ProtectedRoute>
            <Outlet />
          </ProtectedRoute>
        ),
        children: [
          {
            index: true,
            element: <ProductsPage />,
          },
          {
            path: ':id',
            element: <ProductDetailPage />,
            loader: async ({ params }) => {
              // Optional: Preload product data
              return { productId: params.id };
            },
          },
          {
            path: 'create',
            element: <CreateProductPage />,
          },
          {
            path: ':id/edit',
            element: <EditProductPage />,
          },
        ],
      },
      {
        path: 'profile',
        element: (
          <ProtectedRoute>
            <ProfilePage />
          </ProtectedRoute>
        ),
      },
      {
        path: 'admin',
        element: (
          <ProtectedRoute requiredRole="admin">
            <Outlet />
          </ProtectedRoute>
        ),
        children: [
          {
            index: true,
            element: <AdminDashboard />,
          },
          {
            path: 'users',
            element: <UsersManagement />,
          },
          {
            path: 'settings',
            element: <AdminSettings />,
          },
        ],
      },
      {
        path: '404',
        element: <NotFoundPage />,
      },
      {
        path: '*',
        element: <Navigate to="/404" replace />,
      },
    ],
  },
]);
```

#### Protected Route Components
```typescript
// components/auth/ProtectedRoute.tsx
import React from 'react';
import { Navigate, useLocation } from 'react-router-dom';
import { useAppSelector } from '@/app/hooks';
import { CircularProgress, Box } from '@mui/material';

interface ProtectedRouteProps {
  children: React.ReactNode;
  requiredRole?: 'user' | 'admin';
}

export const ProtectedRoute: React.FC<ProtectedRouteProps> = ({ 
  children, 
  requiredRole 
}) => {
  const location = useLocation();
  const { user, token, isLoading } = useAppSelector((state) => state.auth);

  if (isLoading) {
    return (
      <Box className="flex justify-center items-center min-h-64">
        <CircularProgress />
      </Box>
    );
  }

  if (!token || !user) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  if (requiredRole && user.role !== requiredRole) {
    return <Navigate to="/403" replace />;
  }

  return <>{children}</>;
};

// components/auth/GuestRoute.tsx
export const GuestRoute: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const isAuthenticated = useAppSelector((state) => !!state.auth.token);
  
  if (isAuthenticated) {
    return <Navigate to="/products" replace />;
  }

  return <>{children}</>;
};
```

#### App Integration with Router Configuration
```typescript
// App.tsx
import React from 'react';
import { RouterProvider } from 'react-router-dom';
import { ThemeProvider } from '@mui/material/styles';
import { CssBaseline } from '@mui/material';
import { Provider } from 'react-redux';
import { store } from './app/store';
import { theme } from './lib/theme';
import { router } from './lib/router';
import { GlobalLoading } from './components/common/GlobalLoading';

function App() {
  return (
    <Provider store={store}>
      <ThemeProvider theme={theme}>
        <CssBaseline />
        <RouterProvider 
          router={router}
          fallbackElement={<GlobalLoading />}
        />
      </ThemeProvider>
    </Provider>
  );
}

export default App;
```

#### Advanced Router Features
```typescript
// lib/routerUtils.ts
import { redirect } from 'react-router-dom';
import { store } from '@/app/store';

// Route guards
export const requireAuth = () => {
  const state = store.getState();
  if (!state.auth.token) {
    throw redirect('/login');
  }
  return null;
};

export const requireRole = (role: 'admin' | 'user') => () => {
  const state = store.getState();
  if (!state.auth.token) {
    throw redirect('/login');
  }
  if (state.auth.user?.role !== role) {
    throw redirect('/403');
  }
  return null;
};

// Data loaders
export const productLoader = async ({ params }: { params: any }) => {
  try {
    const response = await fetch(`/api/products/${params.id}`);
    if (!response.ok) {
      throw new Response('Product not found', { status: 404 });
    }
    return response.json();
  } catch (error) {
    throw new Response('Failed to load product', { status: 500 });
  }
};

// Usage in router configuration
export const routerWithLoaders = createBrowserRouter([
  {
    path: '/products/:id',
    element: <ProductDetailPage />,
    loader: productLoader,
    action: async ({ request, params }) => {
      // Handle form submissions
      const formData = await request.formData();
      // Process form data
      return redirect('/products');
    },
  },
]);
```

#### Route-based Code Splitting
```typescript
// lib/lazyRoutes.tsx
import { lazy } from 'react';

// Lazy load components for better performance
export const HomePage = lazy(() => import('@/pages/HomePage'));
export const ProductsPage = lazy(() => import('@/pages/ProductsPage'));
export const ProductDetailPage = lazy(() => import('@/pages/ProductDetailPage'));
export const AdminDashboard = lazy(() => import('@/pages/admin/Dashboard'));

// Error boundaries for lazy routes
export const LazyWrapper: React.FC<{ children: React.ReactNode }> = ({ children }) => (
  <Suspense 
    fallback={
      <Box className="flex justify-center items-center min-h-64">
        <CircularProgress />
      </Box>
    }
  >
    <ErrorBoundary>
      {children}
    </ErrorBoundary>
  </Suspense>
);

// Updated router with lazy loading
export const router = createBrowserRouter([
  {
    path: '/',
    element: <Layout />,
    children: [
      {
        index: true,
        element: (
          <LazyWrapper>
            <HomePage />
          </LazyWrapper>
        ),
      },
      // ... other routes
    ],
  },
]);
```

#### Navigation Utilities and Hooks
```typescript
// hooks/useNavigation.ts
import { useNavigate, useLocation, useSearchParams } from 'react-router-dom';
import { useCallback } from 'react';

export const useAppNavigation = () => {
  const navigate = useNavigate();
  const location = useLocation();
  const [searchParams, setSearchParams] = useSearchParams();

  const navigateWithState = useCallback((
    to: string, 
    state?: any, 
    options?: { replace?: boolean }
  ) => {
    navigate(to, { state, replace: options?.replace });
  }, [navigate]);

  const goBack = useCallback(() => {
    navigate(-1);
  }, [navigate]);

  const goToProducts = useCallback((params?: Record<string, string>) => {
    const url = new URL('/products', window.location.origin);
    if (params) {
      Object.entries(params).forEach(([key, value]) => {
        url.searchParams.set(key, value);
      });
    }
    navigate(url.pathname + url.search);
  }, [navigate]);

  const updateSearchParams = useCallback((
    updates: Record<string, string | null>
  ) => {
    const newParams = new URLSearchParams(searchParams);
    
    Object.entries(updates).forEach(([key, value]) => {
      if (value === null) {
        newParams.delete(key);
      } else {
        newParams.set(key, value);
      }
    });

    setSearchParams(newParams);
  }, [searchParams, setSearchParams]);

  return {
    navigate: navigateWithState,
    goBack,
    goToProducts,
    updateSearchParams,
    currentPath: location.pathname,
    currentSearch: location.search,
    searchParams,
  };
};

// hooks/useBreadcrumbs.ts
export const useBreadcrumbs = () => {
  const location = useLocation();
  
  const generateBreadcrumbs = useCallback(() => {
    const pathSegments = location.pathname.split('/').filter(Boolean);
    const breadcrumbs = [{ label: 'Home', path: '/' }];
    
    let currentPath = '';
    pathSegments.forEach((segment, index) => {
      currentPath += `/${segment}`;
      const label = segment.charAt(0).toUpperCase() + segment.slice(1);
      breadcrumbs.push({ 
        label, 
        path: currentPath,
        isActive: index === pathSegments.length - 1
      });
    });
    
    return breadcrumbs;
  }, [location.pathname]);

  return generateBreadcrumbs();
};
```

#### Route-based Data Fetching with RTK Query
```typescript
// lib/routerWithData.tsx
import { router } from './router';
import { store } from '@/app/store';
import { productsApi } from '@/features/products/services/productsApi';

// Enhanced router with data prefetching
export const createDataRouter = () => {
  return createBrowserRouter([
    {
      path: '/products',
      element: <ProductsPage />,
      loader: async () => {
        // Prefetch products data
        const promise = store.dispatch(
          productsApi.endpoints.getProducts.initiate({ page: 1, limit: 12 })
        );
        
        try {
          await promise;
        } catch (error) {
          console.error('Failed to prefetch products:', error);
        } finally {
          promise.unsubscribe();
        }
        
        return null;
      },
    },
    {
      path: '/products/:id',
      element: <ProductDetailPage />,
      loader: async ({ params }) => {
        if (!params.id) {
          throw new Response('Product ID is required', { status: 400 });
        }

        // Prefetch product data
        const promise = store.dispatch(
          productsApi.endpoints.getProductById.initiate(params.id)
        );
        
        try {
          const result = await promise;
          if ('error' in result) {
            throw new Response('Product not found', { status: 404 });
          }
          return { productId: params.id };
        } catch (error) {
          throw new Response('Failed to load product', { status: 500 });
        } finally {
          promise.unsubscribe();
        }
      },
    },
  ]);
};
```

#### Form Component with Redux Integration
```typescript
// features/auth/components/LoginForm.tsx
import React from 'react';
import { useForm, Controller } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { 
  TextField, 
  Button, 
  Box, 
  Alert, 
  Paper,
  Typography 
} from '@mui/material';
import { useAppDispatch, useAppSelector } from '@/app/hooks';
import { loginUser, clearError } from '../slice';
import { useNavigate } from 'react-router-dom';

const loginSchema = z.object({
  email: z.string().email('Invalid email address'),
  password: z.string().min(6, 'Password must be at least 6 characters'),
});

type LoginFormData = z.infer<typeof loginSchema>;

export const LoginForm: React.FC = () => {
  const dispatch = useAppDispatch();
  const navigate = useNavigate();
  const { isLoading, error } = useAppSelector((state) => state.auth);

  const { control, handleSubmit, formState: { errors } } = useForm<LoginFormData>({
    resolver: zodResolver(loginSchema),
    defaultValues: {
      email: '',
      password: '',
    },
  });

  const onSubmit = async (data: LoginFormData) => {
    try {
      await dispatch(loginUser(data)).unwrap();
      navigate('/products');
    } catch (error) {
      // Error is handled by the slice
    }
  };

  React.useEffect(() => {
    return () => {
      dispatch(clearError());
    };
  }, [dispatch]);

  return (
    <Paper className="max-w-md mx-auto mt-8 p-6">
      <Typography variant="h4" component="h1" className="text-center mb-6">
        Login
      </Typography>
      
      {error && (
        <Alert severity="error" className="mb-4">
          {error}
        </Alert>
      )}

      <Box component="form" onSubmit={handleSubmit(onSubmit)} className="space-y-4">
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
            />
          )}
        />
        
        <Controller
          name="password"
          control={control}
          render={({ field }) => (
            <TextField
              {...field}
              label="Password"
              type="password"
              fullWidth
              error={!!errors.password}
              helperText={errors.password?.message}
            />
          )}
        />
        
        <Button
          type="submit"
          variant="contained"
          fullWidth
          disabled={isLoading}
          className="mt-6"
        >
          {isLoading ? 'Logging in...' : 'Login'}
        </Button>
      </Box>
    </Paper>
  );
};
```

### 6. Custom Hooks with Redux

#### Feature-Specific Hook
```typescript
// features/products/hooks/useProducts.ts
import { useMemo } from 'react';
import { useGetProductsQuery } from '../services/productsApi';
import { useAppSelector } from '@/app/hooks';

interface UseProductsOptions {
  page?: number;
  limit?: number;
  category?: string;
  search?: string;
}

export const useProducts = (options: UseProductsOptions = {}) => {
  const { page = 1, limit = 10, category, search } = options;
  
  const queryParams = useMemo(() => ({
    page,
    limit,
    ...(category && { category }),
    ...(search && { search }),
  }), [page, limit, category, search]);

  const {
    data,
    isLoading,
    isError,
    error,
    refetch,
  } = useGetProductsQuery(queryParams);

  const filteredProducts = useMemo(() => {
    if (!data?.products) return [];
    
    let filtered = [...data.products];
    
    if (search) {
      filtered = filtered.filter(product =>
        product.name.toLowerCase().includes(search.toLowerCase()) ||
        product.description.toLowerCase().includes(search.toLowerCase())
      );
    }
    
    return filtered;
  }, [data?.products, search]);

  return {
    products: filteredProducts,
    total: data?.total || 0,
    isLoading,
    isError,
    error,
    refetch,
  };
};
```

### 7. Error Handling and Loading States

#### Error Boundary Component
```typescript
// components/common/ErrorBoundary.tsx
import React from 'react';
import { Alert, Button, Box, Typography } from '@mui/material';
import { ErrorBoundary as ReactErrorBoundary } from 'react-error-boundary';

interface ErrorFallbackProps {
  error: Error;
  resetErrorBoundary: () => void;
}

const ErrorFallback: React.FC<ErrorFallbackProps> = ({ error, resetErrorBoundary }) => {
  return (
    <Box className="flex flex-col items-center justify-center min-h-96 p-6">
      <Alert severity="error" className="mb-4 max-w-md">
        <Typography variant="h6" component="div" className="mb-2">
          Something went wrong
        </Typography>
        <Typography variant="body2" className="mb-4">
          {error.message}
        </Typography>
        <Button onClick={resetErrorBoundary} variant="outlined" size="small">
          Try again
        </Button>
      </Alert>
    </Box>
  );
};

export const ErrorBoundary: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  return (
    <ReactErrorBoundary
      FallbackComponent={ErrorFallback}
      onError={(error, errorInfo) => {
        console.error('Error caught by boundary:', error, errorInfo);
      }}
    >
      {children}
    </ReactErrorBoundary>
  );
};
```

#### Global Loading Component
```typescript
// components/common/GlobalLoading.tsx
import React from 'react';
import { Backdrop, CircularProgress } from '@mui/material';
import { useAppSelector } from '@/app/hooks';

export const GlobalLoading: React.FC = () => {
  const isLoading = useAppSelector((state) => 
    state.auth.isLoading || 
    // Add other loading states from different slices
    Object.values(state).some((slice: any) => slice?.isLoading)
  );

  return (
    <Backdrop open={isLoading} className="z-50">
      <CircularProgress color="primary" />
    </Backdrop>
  );
};
```

### 8. Testing with Redux

#### Component Testing with Redux
```typescript
// features/products/components/__tests__/ProductList.test.tsx
import React from 'react';
import { render, screen, waitFor } from '@testing-library/react';
import { Provider } from 'react-redux';
import { configureStore } from '@reduxjs/toolkit';
import { ThemeProvider } from '@mui/material/styles';
import { BrowserRouter } from 'react-router-dom';
import { ProductList } from '../ProductList';
import { productsApi } from '../../services/productsApi';
import { theme } from '@/lib/theme';

// Mock API response
const mockProducts = [
  { id: '1', name: 'Product 1', description: 'Description 1', price: 100 },
  { id: '2', name: 'Product 2', description: 'Description 2', price: 200 },
];

// Create test store
const createTestStore = (preloadedState = {}) => {
  return configureStore({
    reducer: {
      [productsApi.reducerPath]: productsApi.reducer,
    },
    middleware: (getDefaultMiddleware) =>
      getDefaultMiddleware().concat(productsApi.middleware),
    preloadedState,
  });
};

// Test wrapper component
const TestWrapper: React.FC<{ children: React.ReactNode; store?: any }> = ({ 
  children, 
  store = createTestStore() 
}) => {
  return (
    <Provider store={store}>
      <ThemeProvider theme={theme}>
        <BrowserRouter>
          {children}
        </BrowserRouter>
      </ThemeProvider>
    </Provider>
  );
};

describe('ProductList', () => {
  beforeEach(() => {
    // Mock the API call
    jest.spyOn(productsApi.endpoints.getProducts, 'useQuery').mockReturnValue({
      data: { products: mockProducts, total: 2 },
      isLoading: false,
      isError: false,
      error: null,
      refetch: jest.fn(),
    } as any);
  });

  afterEach(() => {
    jest.restoreAllMocks();
  });

  it('renders product list successfully', async () => {
    render(
      <TestWrapper>
        <ProductList />
      </TestWrapper>
    );

    await waitFor(() => {
      expect(screen.getByText('Product 1')).toBeInTheDocument();
      expect(screen.getByText('Product 2')).toBeInTheDocument();
    });
  });

  it('shows loading state', () => {
    jest.spyOn(productsApi.endpoints.getProducts, 'useQuery').mockReturnValue({
      data: undefined,
      isLoading: true,
      isError: false,
      error: null,
      refetch: jest.fn(),
    } as any);

    render(
      <TestWrapper>
        <ProductList />
      </TestWrapper>
    );

    expect(screen.getByRole('progressbar')).toBeInTheDocument();
  });
});
```

### 9. Middleware and Persistence

#### Redux Persist Setup
```typescript
// app/store.ts (with persistence)
import { configureStore, combineReducers } from '@reduxjs/toolkit';
import { persistStore, persistReducer } from 'redux-persist';
import storage from 'redux-persist/lib/storage';
import { authSlice } from '@/features/auth/slice';
import { productsApi } from '@/features/products/services/productsApi';

const persistConfig = {
  key: 'root',
  storage,
  whitelist: ['auth'], // Only persist auth state
};

const rootReducer = combineReducers({
  auth: authSlice.reducer,
  [productsApi.reducerPath]: productsApi.reducer,
});

const persistedReducer = persistReducer(persistConfig, rootReducer);

export const store = configureStore({
  reducer: persistedReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ['persist/PERSIST', 'persist/REHYDRATE'],
      },
    }).concat(productsApi.middleware),
  devTools: process.env.NODE_ENV !== 'production',
});

export const persistor = persistStore(store);
export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

## Performance Optimization

### 1. Component Memoization
```typescript
// Memoized component to prevent unnecessary re-renders
import React, { memo } from 'react';
import { Product } from '../types';

interface ProductCardProps {
  product: Product;
  onEdit?: (product: Product) => void;
}

export const ProductCard = memo<ProductCardProps>(({ product, onEdit }) => {
  const handleEdit = () => {
    onEdit?.(product);
  };

  return (
    <Card className="h-full">
      <CardContent>
        <Typography variant="h6">{product.name}</Typography>
        <Typography variant="body2" color="text.secondary">
          {product.description}
        </Typography>
        <Typography variant="h6" className="mt-2">
          ${product.price}
        </Typography>
        {onEdit && (
          <Button onClick={handleEdit} size="small" className="mt-2">
            Edit
          </Button>
        )}
      </CardContent>
    </Card>
  );
});

ProductCard.displayName = 'ProductCard';
```

### 2. Selector Optimization
```typescript
// features/products/selectors.ts
import { createSelector } from '@reduxjs/toolkit';
import { RootState } from '@/app/store';

// Memoized selectors for better performance
export const selectProductsState = (state: RootState) => state.products;

export const selectFilteredProducts = createSelector(
  [
    selectProductsState,
    (state: RootState, category: string) => category,
    (state: RootState, category: string, search: string) => search,
  ],
  (productsState, category, search) => {
    let filtered = productsState.items;

    if (category) {
      filtered = filtered.filter(product => product.category === category);
    }

    if (search) {
      filtered = filtered.filter(product =>
        product.name.toLowerCase().includes(search.toLowerCase())
      );
    }

    return filtered;
  }
);
```

## Security Best Practices

### 1. Token Management
```typescript
// lib/tokenManager.ts
class TokenManager {
  private static readonly TOKEN_KEY = 'auth_token';
  private static readonly REFRESH_TOKEN_KEY = 'refresh_token';

  static setTokens(token: string, refreshToken?: string) {
    localStorage.setItem(this.TOKEN_KEY, token);
    if (refreshToken) {
      localStorage.setItem(this.REFRESH_TOKEN_KEY, refreshToken);
    }
  }

  static getToken(): string | null {
    return localStorage.getItem(this.TOKEN_KEY);
  }

  static getRefreshToken(): string | null {
    return localStorage.getItem(this.REFRESH_TOKEN_KEY);
  }

  static clearTokens() {
    localStorage.removeItem(this.TOKEN_KEY);
    localStorage.removeItem(this.REFRESH_TOKEN_KEY);
  }

  static isTokenExpired(token: string): boolean {
    try {
      const payload = JSON.parse(atob(token.split('.')[1]));
      return Date.now() >= payload.exp * 1000;
    } catch {
      return true;
    }
  }
}

export { TokenManager };
```

### 2. Input Validation
```typescript
// lib/validations.ts
import { z } from 'zod';

export const userValidation = z.object({
  id: z.string().uuid(),
  email: z.string().email().max(255),
  name: z.string().min(2).max(100),
  role: z.enum(['user', 'admin']),
});

export const productValidation = z.object({
  name: z.string().min(2).max(100),
  description: z.string().min(10).max(1000),
  price: z.number().positive().max(999999),
  category: z.string().min(2).max(50),
  images: z.array(z.string().url()).max(10),
});

// Sanitize HTML content
import DOMPurify from 'isomorphic-dompurify';

export const sanitizeHTML = (html: string): string => {
  return DOMPurify.sanitize(html);
};
```

## Build and Deployment

### 1. Vite Configuration
```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  build: {
    outDir: 'dist',
    sourcemap: true,
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          redux: ['@reduxjs/toolkit', 'react-redux'],
          ui: ['@mui/material', '@mui/icons-material'],
        },
      },
    },
  },
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true,
      },
    },
  },
});
```

### 2. Environment Configuration
```typescript
// lib/env.ts
import { z } from 'zod';

const envSchema = z.object({
  VITE_API_BASE_URL: z.string().url(),
  VITE_APP_NAME: z.string().min(1),
  VITE_APP_VERSION: z.string().min(1),
  VITE_ENABLE_MSW: z.string().optional(),
});

export const env = envSchema.parse(import.meta.env);
```

## AI Generation Guidelines

When generating React + Redux Toolkit + TypeScript code:

1. **Always use TypeScript** with explicit types and interfaces
2. **Use Redux Toolkit** for state management, never plain Redux
3. **Implement RTK Query** for data fetching and caching
4. **Follow feature-based architecture** with co-located files
5. **Use typed Redux hooks** (useAppDispatch, useAppSelector)
6. **Implement proper error handling** with error boundaries
7. **Include loading states** for all async operations
8. **Use MUI components** enhanced with Tailwind utilities
9. **Validate all inputs** with Zod schemas
10. **Write testable components** with proper mocking
11. **Implement proper routing** with React Router
12. **Use modern React patterns** (hooks, functional components)
13. **Optimize performance** with memoization and selectors
14. **Follow security best practices** for authentication

### Do Not Generate
- Class components (use functional components)
- Plain Redux code (use Redux Toolkit)
- Inline styles (use MUI + Tailwind)
- Untyped Redux code
- Global state for local component state
- Custom CSS files (use MUI theme + Tailwind)
- Example/demo files unless explicitly requested
- Hardcoded API URLs (use environment variables)
