---
applyTo: '**/*'
description: 'Project context'
---

# Environment

Project environment context

## Main packages and libraries

- package manager: `yarn`
- base: `React + TS`
- bundler: `Vite`
- linter: `ESLint`
- styles: `MUI` with `styled-components`
- state manager: `Redux Toolkit`

## Folders

## Project Structure

```
dist/                             # Build directory
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

## Commands

- run local: `yarn dev`
- build the app: `yarn build`
- deploy the app: `yarn deploy`