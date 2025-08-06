---
description: 'React Native development standards and instructions'
applyTo: '**/*.tsx, **/*.ts, **/*.jsx, **/*.js, **/*.json, **/*.native.js, **/*.android.js, **/*.ios.js'
---

# React Native + Redux Toolkit + TypeScript Development Instructions

Comprehensive instructions for AI tools to generate high-quality, production-ready React Native applications using Redux Toolkit for state management and the most popular modern stack.

## Modern Tech Stack

### Core Framework
- **React Native 0.73+** with New Architecture (Fabric & TurboModules)
- **TypeScript 5+** with strict mode
- **Expo SDK 50+** for managed workflow
- **Redux Toolkit (RTK)** for state management
- **React Navigation 6+** for navigation

### Essential Dependencies
- **@reduxjs/toolkit** - Redux Toolkit for state management
- **react-redux** - React bindings for Redux
- **@reduxjs/toolkit/query/react** - RTK Query for data fetching
- **@react-navigation/native** - Navigation foundation
- **@react-navigation/native-stack** - Native stack navigator
- **@react-navigation/bottom-tabs** - Tab navigation
- **@react-navigation/drawer** - Drawer navigation
- **react-native-safe-area-context** - Safe area handling
- **react-native-screens** - Native screen management
- **zod** - Runtime type validation
- **react-hook-form** - Form management
- **@hookform/resolvers** - Form validation resolvers

### UI and Styling
- **NativeWind** - Tailwind CSS for React Native
- **react-native-reanimated 3** - Advanced animations
- **react-native-gesture-handler** - Touch gesture system
- **react-native-vector-icons** - Icon library
- **react-native-paper** - Material Design components
- **react-native-elements** - Cross-platform UI toolkit
- **styled-components/native** - CSS-in-JS styling

### Native Features
- **@react-native-async-storage/async-storage** - Persistent storage
- **react-native-permissions** - Permission handling
- **react-native-image-picker** - Camera/gallery access
- **@react-native-community/netinfo** - Network status
- **react-native-device-info** - Device information
- **react-native-keychain** - Secure storage
- **react-native-biometrics** - Biometric authentication

### Development Tools
- **Metro** - React Native bundler
- **Flipper** - Debugging platform
- **ESLint** with React Native config
- **Prettier** with React Native plugin
- **TypeScript ESLint** rules
- **Husky** + **lint-staged** for git hooks
- **@testing-library/react-native** - Component testing
- **Jest** - Testing framework
- **Detox** - E2E testing

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
│   │   ├── screens/              # Feature screens
│   │   ├── hooks/                # Feature-specific hooks
│   │   ├── services/             # RTK Query API slices
│   │   ├── slice.ts              # Redux slice
│   │   └── types.ts              # Feature types
│   └── products/                 # Products feature
│       ├── components/
│       ├── screens/
│       ├── hooks/
│       ├── services/
│       ├── slice.ts
│       └── types.ts
├── components/                   # Shared components
│   ├── ui/                       # Base UI components
│   ├── forms/                    # Form components
│   ├── layout/                   # Layout components
│   └── common/                   # Common reusable components
├── screens/                      # Shared screens
├── navigation/                   # Navigation configuration
│   ├── AppNavigator.tsx          # Main navigator
│   ├── AuthNavigator.tsx         # Authentication flow
│   ├── TabNavigator.tsx          # Tab navigation
│   └── types.ts                  # Navigation types
├── hooks/                        # Shared custom hooks
├── services/                     # Shared services
│   ├── api/                      # API configuration
│   ├── storage/                  # Storage utilities
│   ├── permissions/              # Permission handling
│   └── notifications/            # Push notifications
├── utils/                        # Utilities and helpers
│   ├── constants.ts              # App constants
│   ├── helpers.ts                # Helper functions
│   ├── validations.ts            # Zod schemas
│   ├── formatters.ts             # Data formatters
│   └── dimensions.ts             # Screen dimensions
├── types/                        # Shared TypeScript types
├── styles/                       # Global styles and themes
│   ├── themes.ts                 # Theme configuration
│   ├── colors.ts                 # Color palette
│   ├── typography.ts             # Text styles
│   └── spacing.ts                # Spacing constants
├── assets/                       # Static assets
│   ├── images/                   # Image assets
│   ├── icons/                    # Icon assets
│   └── fonts/                    # Custom fonts
├── App.tsx                       # Main App component
└── index.js                      # Entry point
```

## Core Development Principles

### 1. Redux Toolkit Best Practices

#### Store Configuration
```typescript
// app/store.ts
import { configureStore } from '@reduxjs/toolkit';
import { setupListeners } from '@reduxjs/toolkit/query';
import AsyncStorage from '@react-native-async-storage/async-storage';
import { persistStore, persistReducer } from 'redux-persist';
import { authSlice } from '@/features/auth/slice';
import { productsApi } from '@/features/products/services/productsApi';

const persistConfig = {
  key: 'root',
  storage: AsyncStorage,
  whitelist: ['auth'], // Only persist auth state
};

const persistedAuthReducer = persistReducer(persistConfig, authSlice.reducer);

export const store = configureStore({
  reducer: {
    auth: persistedAuthReducer,
    [productsApi.reducerPath]: productsApi.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ['persist/PERSIST', 'persist/REHYDRATE'],
      },
    }).concat(productsApi.middleware),
  devTools: __DEV__,
});

setupListeners(store.dispatch);

export const persistor = persistStore(store);
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

### 2. Navigation Setup

#### Main Navigator
```typescript
// navigation/AppNavigator.tsx
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { useAppSelector } from '@/app/hooks';
import { AuthNavigator } from './AuthNavigator';
import { TabNavigator } from './TabNavigator';
import { LoadingScreen } from '@/components/common/LoadingScreen';
import { RootStackParamList } from './types';

const Stack = createNativeStackNavigator<RootStackParamList>();

export const AppNavigator: React.FC = () => {
  const { isAuthenticated, isLoading } = useAppSelector((state) => state.auth);

  if (isLoading) {
    return <LoadingScreen />;
  }

  return (
    <NavigationContainer>
      <Stack.Navigator screenOptions={{ headerShown: false }}>
        {isAuthenticated ? (
          <Stack.Screen name="Main" component={TabNavigator} />
        ) : (
          <Stack.Screen name="Auth" component={AuthNavigator} />
        )}
      </Stack.Navigator>
    </NavigationContainer>
  );
};
```

#### Tab Navigator
```typescript
// navigation/TabNavigator.tsx
import React from 'react';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import Icon from 'react-native-vector-icons/MaterialIcons';
import { HomeScreen } from '@/screens/HomeScreen';
import { ProductsScreen } from '@/features/products/screens/ProductsScreen';
import { ProductDetailScreen } from '@/features/products/screens/ProductDetailScreen';
import { ProfileScreen } from '@/screens/ProfileScreen';
import { colors } from '@/styles/colors';
import { TabParamList, ProductsStackParamList } from './types';

const Tab = createBottomTabNavigator<TabParamList>();
const ProductsStack = createNativeStackNavigator<ProductsStackParamList>();

const ProductsNavigator = () => {
  return (
    <ProductsStack.Navigator>
      <ProductsStack.Screen 
        name="ProductsList" 
        component={ProductsScreen}
        options={{ title: 'Products' }}
      />
      <ProductsStack.Screen 
        name="ProductDetail" 
        component={ProductDetailScreen}
        options={{ title: 'Product Details' }}
      />
    </ProductsStack.Navigator>
  );
};

export const TabNavigator: React.FC = () => {
  return (
    <Tab.Navigator
      screenOptions={({ route }) => ({
        tabBarIcon: ({ focused, color, size }) => {
          let iconName: string;

          switch (route.name) {
            case 'Home':
              iconName = 'home';
              break;
            case 'Products':
              iconName = 'shopping-cart';
              break;
            case 'Profile':
              iconName = 'person';
              break;
            default:
              iconName = 'circle';
          }

          return <Icon name={iconName} size={size} color={color} />;
        },
        tabBarActiveTintColor: colors.primary,
        tabBarInactiveTintColor: colors.gray,
        headerShown: false,
      })}
    >
      <Tab.Screen name="Home" component={HomeScreen} />
      <Tab.Screen name="Products" component={ProductsNavigator} />
      <Tab.Screen name="Profile" component={ProfileScreen} />
    </Tab.Navigator>
  );
};
```

#### Navigation Types
```typescript
// navigation/types.ts
import type { NavigatorScreenParams } from '@react-navigation/native';
import type { NativeStackScreenProps } from '@react-navigation/native-stack';
import type { BottomTabScreenProps } from '@react-navigation/bottom-tabs';

export type RootStackParamList = {
  Auth: NavigatorScreenParams<AuthStackParamList>;
  Main: NavigatorScreenParams<TabParamList>;
};

export type AuthStackParamList = {
  Login: undefined;
  Register: undefined;
  ForgotPassword: undefined;
};

export type TabParamList = {
  Home: undefined;
  Products: NavigatorScreenParams<ProductsStackParamList>;
  Profile: undefined;
};

export type ProductsStackParamList = {
  ProductsList: undefined;
  ProductDetail: { productId: string };
  CreateProduct: undefined;
  EditProduct: { productId: string };
};

// Screen props types
export type RootStackScreenProps<T extends keyof RootStackParamList> =
  NativeStackScreenProps<RootStackParamList, T>;

export type TabScreenProps<T extends keyof TabParamList> =
  BottomTabScreenProps<TabParamList, T>;

export type ProductsStackScreenProps<T extends keyof ProductsStackParamList> =
  NativeStackScreenProps<ProductsStackParamList, T>;
```

### 3. Component Architecture

#### Screen Component Example
```typescript
// features/products/screens/ProductsScreen.tsx
import React, { useState, useCallback } from 'react';
import {
  View,
  FlatList,
  RefreshControl,
  ActivityIndicator,
  Text,
  TextInput,
  TouchableOpacity,
} from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import { useNavigation } from '@react-navigation/native';
import Icon from 'react-native-vector-icons/MaterialIcons';
import { useGetProductsQuery } from '../services/productsApi';
import { ProductCard } from '../components/ProductCard';
import { ErrorMessage } from '@/components/common/ErrorMessage';
import { EmptyState } from '@/components/common/EmptyState';
import { colors } from '@/styles/colors';
import { spacing } from '@/styles/spacing';
import type { ProductsStackScreenProps } from '@/navigation/types';

type Props = ProductsStackScreenProps<'ProductsList'>;

export const ProductsScreen: React.FC<Props> = ({ navigation }) => {
  const [search, setSearch] = useState('');
  const [page, setPage] = useState(1);

  const {
    data: productsData,
    isLoading,
    isError,
    error,
    refetch,
    isFetching,
  } = useGetProductsQuery({ page, limit: 20, search });

  const handleProductPress = useCallback((productId: string) => {
    navigation.navigate('ProductDetail', { productId });
  }, [navigation]);

  const handleRefresh = useCallback(() => {
    setPage(1);
    refetch();
  }, [refetch]);

  const handleLoadMore = useCallback(() => {
    if (productsData?.hasNextPage && !isFetching) {
      setPage(prev => prev + 1);
    }
  }, [productsData?.hasNextPage, isFetching]);

  const renderProductItem = useCallback(({ item }) => (
    <ProductCard
      product={item}
      onPress={() => handleProductPress(item.id)}
    />
  ), [handleProductPress]);

  const renderListHeader = () => (
    <View className="px-4 pb-4">
      <View className="flex-row items-center bg-gray-100 rounded-lg px-3 py-2">
        <Icon name="search" size={20} color={colors.gray} />
        <TextInput
          className="flex-1 ml-2 text-base"
          placeholder="Search products..."
          value={search}
          onChangeText={setSearch}
          returnKeyType="search"
        />
        {search.length > 0 && (
          <TouchableOpacity onPress={() => setSearch('')}>
            <Icon name="clear" size={20} color={colors.gray} />
          </TouchableOpacity>
        )}
      </View>
    </View>
  );

  const renderListFooter = () => {
    if (!isFetching || page === 1) return null;
    
    return (
      <View className="py-4">
        <ActivityIndicator size="small" color={colors.primary} />
      </View>
    );
  };

  if (isLoading && page === 1) {
    return (
      <SafeAreaView className="flex-1 justify-center items-center">
        <ActivityIndicator size="large" color={colors.primary} />
        <Text className="mt-2 text-gray-600">Loading products...</Text>
      </SafeAreaView>
    );
  }

  if (isError) {
    return (
      <SafeAreaView className="flex-1">
        <ErrorMessage
          message={error?.message || 'Failed to load products'}
          onRetry={refetch}
        />
      </SafeAreaView>
    );
  }

  return (
    <SafeAreaView className="flex-1 bg-white">
      <FlatList
        data={productsData?.products || []}
        renderItem={renderProductItem}
        keyExtractor={(item) => item.id}
        numColumns={2}
        columnWrapperStyle={{ paddingHorizontal: spacing.md }}
        contentContainerStyle={{ paddingVertical: spacing.sm }}
        ListHeaderComponent={renderListHeader}
        ListFooterComponent={renderListFooter}
        ListEmptyComponent={
          <EmptyState
            icon="shopping-cart"
            title="No products found"
            message="Try adjusting your search or check back later"
          />
        }
        refreshControl={
          <RefreshControl
            refreshing={isLoading && page === 1}
            onRefresh={handleRefresh}
            colors={[colors.primary]}
          />
        }
        onEndReached={handleLoadMore}
        onEndReachedThreshold={0.5}
        showsVerticalScrollIndicator={false}
      />
    </SafeAreaView>
  );
};
```

#### Reusable Component Example
```typescript
// features/products/components/ProductCard.tsx
import React, { memo } from 'react';
import {
  View,
  Text,
  Image,
  TouchableOpacity,
  Dimensions,
} from 'react-native';
import Icon from 'react-native-vector-icons/MaterialIcons';
import { Product } from '../types';
import { colors } from '@/styles/colors';
import { spacing } from '@/styles/spacing';

interface ProductCardProps {
  product: Product;
  onPress: () => void;
  onFavorite?: () => void;
  isFavorite?: boolean;
}

const { width } = Dimensions.get('window');
const cardWidth = (width - spacing.md * 3) / 2;

export const ProductCard = memo<ProductCardProps>(({
  product,
  onPress,
  onFavorite,
  isFavorite = false,
}) => {
  return (
    <TouchableOpacity
      className="bg-white rounded-lg shadow-sm mb-4 overflow-hidden"
      style={{ width: cardWidth }}
      onPress={onPress}
      activeOpacity={0.7}
    >
      <View className="relative">
        <Image
          source={{ uri: product.imageUrl }}
          className="w-full h-32"
          resizeMode="cover"
        />
        {onFavorite && (
          <TouchableOpacity
            className="absolute top-2 right-2 w-8 h-8 bg-white rounded-full items-center justify-center"
            onPress={onFavorite}
            hitSlop={{ top: 10, bottom: 10, left: 10, right: 10 }}
          >
            <Icon
              name={isFavorite ? 'favorite' : 'favorite-border'}
              size={20}
              color={isFavorite ? colors.error : colors.gray}
            />
          </TouchableOpacity>
        )}
        {product.discount && (
          <View className="absolute top-2 left-2 bg-red-500 px-2 py-1 rounded">
            <Text className="text-white text-xs font-semibold">
              -{product.discount}%
            </Text>
          </View>
        )}
      </View>
      
      <View className="p-3">
        <Text
          className="text-sm font-semibold text-gray-900 mb-1"
          numberOfLines={2}
        >
          {product.name}
        </Text>
        
        <Text
          className="text-xs text-gray-600 mb-2"
          numberOfLines={1}
        >
          {product.category}
        </Text>
        
        <View className="flex-row items-center justify-between">
          <View className="flex-row items-center">
            <Text className="text-lg font-bold text-gray-900">
              ${product.price}
            </Text>
            {product.originalPrice && (
              <Text className="text-sm text-gray-500 line-through ml-2">
                ${product.originalPrice}
              </Text>
            )}
          </View>
          
          {product.rating && (
            <View className="flex-row items-center">
              <Icon name="star" size={14} color={colors.warning} />
              <Text className="text-xs text-gray-600 ml-1">
                {product.rating.toFixed(1)}
              </Text>
            </View>
          )}
        </View>
      </View>
    </TouchableOpacity>
  );
});

ProductCard.displayName = 'ProductCard';
```

### 4. Form Handling with React Hook Form

#### Authentication Form Example
```typescript
// features/auth/components/LoginForm.tsx
import React from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  Alert,
  KeyboardAvoidingView,
  Platform,
} from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import { useForm, Controller } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { useNavigation } from '@react-navigation/native';
import { TextInput } from '@/components/ui/TextInput';
import { Button } from '@/components/ui/Button';
import { useAppDispatch, useAppSelector } from '@/app/hooks';
import { loginUser } from '../slice';
import type { AuthStackScreenProps } from '@/navigation/types';

const loginSchema = z.object({
  email: z.string().email('Invalid email address'),
  password: z.string().min(6, 'Password must be at least 6 characters'),
});

type LoginFormData = z.infer<typeof loginSchema>;
type Props = AuthStackScreenProps<'Login'>;

export const LoginForm: React.FC<Props> = ({ navigation }) => {
  const dispatch = useAppDispatch();
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
      // Navigation will be handled by AppNavigator based on auth state
    } catch (error) {
      Alert.alert(
        'Login Failed',
        error?.message || 'Please check your credentials and try again.',
        [{ text: 'OK' }]
      );
    }
  };

  const navigateToRegister = () => {
    navigation.navigate('Register');
  };

  const navigateToForgotPassword = () => {
    navigation.navigate('ForgotPassword');
  };

  return (
    <SafeAreaView className="flex-1 bg-white">
      <KeyboardAvoidingView
        className="flex-1"
        behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
      >
        <View className="flex-1 justify-center px-6">
          <View className="mb-8">
            <Text className="text-3xl font-bold text-gray-900 mb-2">
              Welcome Back
            </Text>
            <Text className="text-gray-600">
              Sign in to your account to continue
            </Text>
          </View>

          {error && (
            <View className="bg-red-50 border border-red-200 rounded-lg p-3 mb-4">
              <Text className="text-red-700 text-sm">{error}</Text>
            </View>
          )}

          <View className="space-y-4">
            <Controller
              name="email"
              control={control}
              render={({ field: { onChange, onBlur, value } }) => (
                <TextInput
                  label="Email"
                  placeholder="Enter your email"
                  value={value}
                  onChangeText={onChange}
                  onBlur={onBlur}
                  error={errors.email?.message}
                  keyboardType="email-address"
                  autoCapitalize="none"
                  autoComplete="email"
                />
              )}
            />

            <Controller
              name="password"
              control={control}
              render={({ field: { onChange, onBlur, value } }) => (
                <TextInput
                  label="Password"
                  placeholder="Enter your password"
                  value={value}
                  onChangeText={onChange}
                  onBlur={onBlur}
                  error={errors.password?.message}
                  secureTextEntry
                  autoComplete="password"
                />
              )}
            />
          </View>

          <TouchableOpacity
            className="self-end mt-2"
            onPress={navigateToForgotPassword}
          >
            <Text className="text-blue-600 font-medium">
              Forgot Password?
            </Text>
          </TouchableOpacity>

          <Button
            title="Sign In"
            onPress={handleSubmit(onSubmit)}
            loading={isLoading}
            className="mt-6"
          />

          <View className="flex-row justify-center items-center mt-6">
            <Text className="text-gray-600">Don't have an account? </Text>
            <TouchableOpacity onPress={navigateToRegister}>
              <Text className="text-blue-600 font-medium">Sign Up</Text>
            </TouchableOpacity>
          </View>
        </View>
      </KeyboardAvoidingView>
    </SafeAreaView>
  );
};
```

### 5. Custom Hooks

#### Network Status Hook
```typescript
// hooks/useNetworkStatus.ts
import { useState, useEffect } from 'react';
import NetInfo from '@react-native-community/netinfo';

export interface NetworkStatus {
  isConnected: boolean;
  isInternetReachable: boolean;
  type: string | null;
}

export const useNetworkStatus = () => {
  const [networkStatus, setNetworkStatus] = useState<NetworkStatus>({
    isConnected: true,
    isInternetReachable: true,
    type: null,
  });

  useEffect(() => {
    const unsubscribe = NetInfo.addEventListener(state => {
      setNetworkStatus({
        isConnected: state.isConnected ?? false,
        isInternetReachable: state.isInternetReachable ?? false,
        type: state.type,
      });
    });

    return unsubscribe;
  }, []);

  return networkStatus;
};
```

#### Permissions Hook
```typescript
// hooks/usePermissions.ts
import { useState, useEffect } from 'react';
import {
  request,
  check,
  PERMISSIONS,
  RESULTS,
  Permission,
} from 'react-native-permissions';
import { Platform } from 'react-native';

type PermissionStatus = 'granted' | 'denied' | 'blocked' | 'unavailable' | 'limited';

export const usePermissions = (permission: Permission) => {
  const [status, setStatus] = useState<PermissionStatus>('unavailable');
  const [isLoading, setIsLoading] = useState(true);

  const checkPermission = async () => {
    try {
      const result = await check(permission);
      setStatus(result as PermissionStatus);
    } catch (error) {
      console.error('Permission check error:', error);
      setStatus('unavailable');
    } finally {
      setIsLoading(false);
    }
  };

  const requestPermission = async (): Promise<PermissionStatus> => {
    try {
      setIsLoading(true);
      const result = await request(permission);
      setStatus(result as PermissionStatus);
      return result as PermissionStatus;
    } catch (error) {
      console.error('Permission request error:', error);
      setStatus('denied');
      return 'denied';
    } finally {
      setIsLoading(false);
    }
  };

  useEffect(() => {
    checkPermission();
  }, [permission]);

  return {
    status,
    isLoading,
    isGranted: status === 'granted',
    isDenied: status === 'denied',
    isBlocked: status === 'blocked',
    checkPermission,
    requestPermission,
  };
};

// Usage examples
export const useCameraPermission = () => {
  return usePermissions(
    Platform.OS === 'ios' 
      ? PERMISSIONS.IOS.CAMERA 
      : PERMISSIONS.ANDROID.CAMERA
  );
};

export const useLocationPermission = () => {
  return usePermissions(
    Platform.OS === 'ios'
      ? PERMISSIONS.IOS.LOCATION_WHEN_IN_USE
      : PERMISSIONS.ANDROID.ACCESS_FINE_LOCATION
  );
};
```

### 6. UI Components

#### Reusable TextInput Component
```typescript
// components/ui/TextInput.tsx
import React, { forwardRef } from 'react';
import {
  View,
  Text,
  TextInput as RNTextInput,
  TextInputProps as RNTextInputProps,
  TouchableOpacity,
} from 'react-native';
import Icon from 'react-native-vector-icons/MaterialIcons';
import { colors } from '@/styles/colors';

interface TextInputProps extends RNTextInputProps {
  label?: string;
  error?: string;
  leftIcon?: string;
  rightIcon?: string;
  onRightIconPress?: () => void;
  containerClassName?: string;
  inputClassName?: string;
}

export const TextInput = forwardRef<RNTextInput, TextInputProps>(({
  label,
  error,
  leftIcon,
  rightIcon,
  onRightIconPress,
  containerClassName = '',
  inputClassName = '',
  style,
  ...props
}, ref) => {
  const hasError = !!error;

  return (
    <View className={`mb-4 ${containerClassName}`}>
      {label && (
        <Text className="text-gray-700 font-medium mb-2">
          {label}
        </Text>
      )}
      
      <View
        className={`
          flex-row items-center border rounded-lg px-3 py-2
          ${hasError ? 'border-red-500' : 'border-gray-300'}
          ${props.editable === false ? 'bg-gray-100' : 'bg-white'}
        `}
      >
        {leftIcon && (
          <Icon
            name={leftIcon}
            size={20}
            color={hasError ? colors.error : colors.gray}
            style={{ marginRight: 8 }}
          />
        )}
        
        <RNTextInput
          ref={ref}
          className={`flex-1 text-base text-gray-900 ${inputClassName}`}
          placeholderTextColor={colors.gray}
          style={style}
          {...props}
        />
        
        {rightIcon && (
          <TouchableOpacity
            onPress={onRightIconPress}
            hitSlop={{ top: 10, bottom: 10, left: 10, right: 10 }}
          >
            <Icon
              name={rightIcon}
              size={20}
              color={hasError ? colors.error : colors.gray}
            />
          </TouchableOpacity>
        )}
      </View>
      
      {error && (
        <Text className="text-red-500 text-sm mt-1">
          {error}
        </Text>
      )}
    </View>
  );
});

TextInput.displayName = 'TextInput';
```

#### Button Component
```typescript
// components/ui/Button.tsx
import React from 'react';
import {
  TouchableOpacity,
  Text,
  ActivityIndicator,
  TouchableOpacityProps,
} from 'react-native';
import Icon from 'react-native-vector-icons/MaterialIcons';
import { colors } from '@/styles/colors';

interface ButtonProps extends TouchableOpacityProps {
  title: string;
  variant?: 'primary' | 'secondary' | 'outline' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  loading?: boolean;
  disabled?: boolean;
  leftIcon?: string;
  rightIcon?: string;
  fullWidth?: boolean;
}

export const Button: React.FC<ButtonProps> = ({
  title,
  variant = 'primary',
  size = 'md',
  loading = false,
  disabled = false,
  leftIcon,
  rightIcon,
  fullWidth = false,
  className = '',
  ...props
}) => {
  const isDisabled = disabled || loading;

  const getVariantStyles = () => {
    switch (variant) {
      case 'primary':
        return isDisabled
          ? 'bg-gray-300'
          : 'bg-blue-600 active:bg-blue-700';
      case 'secondary':
        return isDisabled
          ? 'bg-gray-200'
          : 'bg-gray-600 active:bg-gray-700';
      case 'outline':
        return isDisabled
          ? 'border border-gray-300 bg-transparent'
          : 'border border-blue-600 bg-transparent active:bg-blue-50';
      case 'ghost':
        return isDisabled
          ? 'bg-transparent'
          : 'bg-transparent active:bg-gray-100';
      default:
        return 'bg-blue-600 active:bg-blue-700';
    }
  };

  const getSizeStyles = () => {
    switch (size) {
      case 'sm':
        return 'px-3 py-2';
      case 'md':
        return 'px-4 py-3';
      case 'lg':
        return 'px-6 py-4';
      default:
        return 'px-4 py-3';
    }
  };

  const getTextColor = () => {
    if (isDisabled) return 'text-gray-500';
    
    switch (variant) {
      case 'primary':
      case 'secondary':
        return 'text-white';
      case 'outline':
        return 'text-blue-600';
      case 'ghost':
        return 'text-gray-700';
      default:
        return 'text-white';
    }
  };

  const getIconColor = () => {
    if (isDisabled) return colors.gray;
    
    switch (variant) {
      case 'primary':
      case 'secondary':
        return colors.white;
      case 'outline':
        return colors.primary;
      case 'ghost':
        return colors.gray;
      default:
        return colors.white;
    }
  };

  return (
    <TouchableOpacity
      className={`
        flex-row items-center justify-center rounded-lg
        ${getVariantStyles()}
        ${getSizeStyles()}
        ${fullWidth ? 'w-full' : 'self-start'}
        ${className}
      `}
      disabled={isDisabled}
      {...props}
    >
      {loading ? (
        <ActivityIndicator
          size="small"
          color={variant === 'outline' || variant === 'ghost' ? colors.primary : colors.white}
        />
      ) : (
        <>
          {leftIcon && (
            <Icon
              name={leftIcon}
              size={20}
              color={getIconColor()}
              style={{ marginRight: 8 }}
            />
          )}
          
          <Text
            className={`
              font-semibold text-base
              ${getTextColor()}
            `}
          >
            {title}
          </Text>
          
          {rightIcon && (
            <Icon
              name={rightIcon}
              size={20}
              color={getIconColor()}
              style={{ marginLeft: 8 }}
            />
          )}
        </>
      )}
    </TouchableOpacity>
  );
};
```

### 7. Animations with Reanimated

#### Animated List Item
```typescript
// components/common/AnimatedListItem.tsx
import React, { useEffect } from 'react';
import { TouchableOpacity } from 'react-native';
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSpring,
  withTiming,
  runOnJS,
} from 'react-native-reanimated';
import { Gesture, GestureDetector } from 'react-native-gesture-handler';

interface AnimatedListItemProps {
  children: React.ReactNode;
  onPress?: () => void;
  onSwipeLeft?: () => void;
  onSwipeRight?: () => void;
  index: number;
}

export const AnimatedListItem: React.FC<AnimatedListItemProps> = ({
  children,
  onPress,
  onSwipeLeft,
  onSwipeRight,
  index,
}) => {
  const translateX = useSharedValue(0);
  const opacity = useSharedValue(0);
  const scale = useSharedValue(0.8);

  useEffect(() => {
    // Staggered entrance animation
    const delay = index * 100;
    
    setTimeout(() => {
      opacity.value = withTiming(1, { duration: 300 });
      scale.value = withSpring(1, { damping: 15 });
    }, delay);
  }, [index]);

  const panGesture = Gesture.Pan()
    .onUpdate((event) => {
      translateX.value = event.translationX;
    })
    .onEnd((event) => {
      const threshold = 100;
      
      if (event.translationX > threshold && onSwipeRight) {
        runOnJS(onSwipeRight)();
      } else if (event.translationX < -threshold && onSwipeLeft) {
        runOnJS(onSwipeLeft)();
      }
      
      translateX.value = withSpring(0);
    });

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [
      { translateX: translateX.value },
      { scale: scale.value },
    ],
    opacity: opacity.value,
  }));

  const pressGesture = Gesture.Tap()
    .onEnd(() => {
      if (onPress) {
        runOnJS(onPress)();
      }
    });

  const combinedGesture = Gesture.Exclusive(panGesture, pressGesture);

  return (
    <GestureDetector gesture={combinedGesture}>
      <Animated.View style={animatedStyle}>
        {children}
      </Animated.View>
    </GestureDetector>
  );
};
```

### 8. Storage and Security

#### Secure Storage Service
```typescript
// services/storage/secureStorage.ts
import AsyncStorage from '@react-native-async-storage/async-storage';
import Keychain from 'react-native-keychain';

class SecureStorageService {
  // Secure storage for sensitive data (tokens, passwords)
  async setSecureItem(key: string, value: string): Promise<void> {
    try {
      await Keychain.setInternetCredentials(key, key, value);
    } catch (error) {
      console.error('Error storing secure item:', error);
      throw new Error('Failed to store secure item');
    }
  }

  async getSecureItem(key: string): Promise<string | null> {
    try {
      const credentials = await Keychain.getInternetCredentials(key);
      if (credentials && credentials.password) {
        return credentials.password;
      }
      return null;
    } catch (error) {
      console.error('Error retrieving secure item:', error);
      return null;
    }
  }

  async removeSecureItem(key: string): Promise<void> {
    try {
      await Keychain.resetInternetCredentials(key);
    } catch (error) {
      console.error('Error removing secure item:', error);
    }
  }

  // Regular storage for non-sensitive data
  async setItem(key: string, value: any): Promise<void> {
    try {
      const jsonValue = JSON.stringify(value);
      await AsyncStorage.setItem(key, jsonValue);
    } catch (error) {
      console.error('Error storing item:', error);
      throw new Error('Failed to store item');
    }
  }

  async getItem<T = any>(key: string): Promise<T | null> {
    try {
      const jsonValue = await AsyncStorage.getItem(key);
      return jsonValue != null ? JSON.parse(jsonValue) : null;
    } catch (error) {
      console.error('Error retrieving item:', error);
      return null;
    }
  }

  async removeItem(key: string): Promise<void> {
    try {
      await AsyncStorage.removeItem(key);
    } catch (error) {
      console.error('Error removing item:', error);
    }
  }

  async clear(): Promise<void> {
    try {
      await AsyncStorage.clear();
      await Keychain.resetInternetCredentials();
    } catch (error) {
      console.error('Error clearing storage:', error);
    }
  }
}

export const secureStorage = new SecureStorageService();

// Storage keys constants
export const STORAGE_KEYS = {
  AUTH_TOKEN: 'auth_token',
  REFRESH_TOKEN: 'refresh_token',
  USER_PREFERENCES: 'user_preferences',
  CACHED_DATA: 'cached_data',
} as const;
```

### 9. Testing

#### Component Testing Example
```typescript
// features/products/components/__tests__/ProductCard.test.tsx
import React from 'react';
import { render, fireEvent } from '@testing-library/react-native';
import { ProductCard } from '../ProductCard';

const mockProduct = {
  id: '1',
  name: 'Test Product',
  price: 99.99,
  imageUrl: 'https://example.com/image.jpg',
  category: 'Electronics',
  rating: 4.5,
};

describe('ProductCard', () => {
  const mockOnPress = jest.fn();
  const mockOnFavorite = jest.fn();

  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('renders product information correctly', () => {
    const { getByText } = render(
      <ProductCard
        product={mockProduct}
        onPress={mockOnPress}
        onFavorite={mockOnFavorite}
      />
    );

    expect(getByText('Test Product')).toBeTruthy();
    expect(getByText('$99.99')).toBeTruthy();
    expect(getByText('Electronics')).toBeTruthy();
    expect(getByText('4.5')).toBeTruthy();
  });

  it('calls onPress when card is pressed', () => {
    const { getByText } = render(
      <ProductCard
        product={mockProduct}
        onPress={mockOnPress}
        onFavorite={mockOnFavorite}
      />
    );

    fireEvent.press(getByText('Test Product'));
    expect(mockOnPress).toHaveBeenCalledTimes(1);
  });

  it('calls onFavorite when favorite button is pressed', () => {
    const { getByTestId } = render(
      <ProductCard
        product={mockProduct}
        onPress={mockOnPress}
        onFavorite={mockOnFavorite}
      />
    );

    const favoriteButton = getByTestId('favorite-button');
    fireEvent.press(favoriteButton);
    expect(mockOnFavorite).toHaveBeenCalledTimes(1);
  });
});
```

### 10. Performance Optimization

#### Image Optimization Component
```typescript
// components/common/OptimizedImage.tsx
import React, { useState } from 'react';
import {
  View,
  Image,
  ImageProps,
  ActivityIndicator,
  Text,
} from 'react-native';
import FastImage, { FastImageProps } from 'react-native-fast-image';
import { colors } from '@/styles/colors';

interface OptimizedImageProps extends Omit<FastImageProps, 'source'> {
  source: { uri: string } | number;
  fallbackSource?: { uri: string } | number;
  showLoader?: boolean;
  showError?: boolean;
  loaderSize?: 'small' | 'large';
  errorText?: string;
}

export const OptimizedImage: React.FC<OptimizedImageProps> = ({
  source,
  fallbackSource,
  showLoader = true,
  showError = true,
  loaderSize = 'small',
  errorText = 'Failed to load image',
  style,
  ...props
}) => {
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(false);

  const handleLoadStart = () => {
    setLoading(true);
    setError(false);
  };

  const handleLoadEnd = () => {
    setLoading(false);
  };

  const handleError = () => {
    setLoading(false);
    setError(true);
  };

  const imageSource = error && fallbackSource ? fallbackSource : source;

  return (
    <View style={style} className="relative">
      <FastImage
        {...props}
        source={imageSource}
        onLoadStart={handleLoadStart}
        onLoadEnd={handleLoadEnd}
        onError={handleError}
        style={[style, { opacity: loading ? 0 : 1 }]}
      />
      
      {loading && showLoader && (
        <View className="absolute inset-0 justify-center items-center bg-gray-100">
          <ActivityIndicator size={loaderSize} color={colors.primary} />
        </View>
      )}
      
      {error && showError && !fallbackSource && (
        <View className="absolute inset-0 justify-center items-center bg-gray-100">
          <Text className="text-gray-500 text-sm text-center px-2">
            {errorText}
          </Text>
        </View>
      )}
    </View>
  );
};
```

## Platform-Specific Considerations

### 1. iOS-Specific Features
```typescript
// utils/platform.ts
import { Platform, Dimensions } from 'react-native';
import DeviceInfo from 'react-native-device-info';

export const isIOS = Platform.OS === 'ios';
export const isAndroid = Platform.OS === 'android';

export const getStatusBarHeight = () => {
  if (isIOS) {
    return DeviceInfo.hasNotch() ? 44 : 20;
  }
  return Platform.Version >= 23 ? 24 : 0;
};

export const getBottomSafeAreaHeight = () => {
  if (isIOS && DeviceInfo.hasNotch()) {
    return 34;
  }
  return 0;
};

// iOS-specific component styling
export const getIOSStyles = () => ({
  shadowOffset: { width: 0, height: 2 },
  shadowOpacity: 0.1,
  shadowRadius: 4,
  elevation: 0, // Android shadow
});

// Android-specific component styling
export const getAndroidStyles = () => ({
  elevation: 4,
  shadowOffset: { width: 0, height: 0 },
  shadowOpacity: 0,
  shadowRadius: 0,
});
```

### 2. Responsive Design
```typescript
// styles/dimensions.ts
import { Dimensions } from 'react-native';

const { width, height } = Dimensions.get('window');

export const SCREEN_WIDTH = width;
export const SCREEN_HEIGHT = height;

export const isTablet = width >= 768;
export const isSmallScreen = width < 375;
export const isLargeScreen = width > 414;

// Responsive spacing
export const getResponsiveSpacing = (base: number) => {
  if (isSmallScreen) return base * 0.8;
  if (isLargeScreen) return base * 1.2;
  return base;
};

// Responsive font sizes
export const getResponsiveFontSize = (size: number) => {
  const scale = width / 375; // Base width (iPhone X)
  const newSize = size * scale;
  
  // Limit the scaling
  return Math.max(12, Math.min(newSize, size * 1.3));
};
```

## AI Generation Guidelines

When generating React Native code:

1. **Always use TypeScript** with explicit types and interfaces
2. **Use Redux Toolkit** for state management, never plain Redux
3. **Implement RTK Query** for data fetching and caching
4. **Follow feature-based architecture** with co-located files
5. **Use typed Redux hooks** (useAppDispatch, useAppSelector)
6. **Implement proper navigation** with React Navigation 6+
7. **Use NativeWind** for styling with Tailwind CSS
8. **Include error handling** with proper error boundaries
9. **Implement loading states** for all async operations
10. **Use proper form validation** with React Hook Form + Zod
11. **Handle platform differences** (iOS/Android specific code)
12. **Implement proper permissions** handling
13. **Use secure storage** for sensitive data
14. **Optimize for performance** with memo, useMemo, useCallback
15. **Follow accessibility guidelines** with proper a11y props
16. **Implement proper animations** with Reanimated 3
17. **Use FlatList/SectionList** for large data sets
18. **Handle network states** and offline scenarios
19. **Implement proper error handling** and user feedback
20. **Use proper image optimization** with FastImage

### Do Not Generate
- Class components (use functional components with hooks)
- Plain Redux code (use Redux Toolkit)
- Inline styles without NativeWind classes
- Untyped navigation or Redux code
- Global state for local component state
- Custom implementations of existing libraries
- Hardcoded dimensions (use responsive utilities)
- Non-accessible components
- Performance-heavy operations in render
- Direct AsyncStorage usage (use secure storage service)

### Always Include
- Proper TypeScript types for all props and state
- Error boundaries for screen components
- Loading states for async operations
- Proper navigation typing
- Platform-specific considerations
- Accessibility props (accessibilityLabel, accessibilityHint)
- Proper memory cleanup in useEffect
- Performance optimizations (memo, useMemo, useCallback)
- Proper image optimization
- Network error handling
