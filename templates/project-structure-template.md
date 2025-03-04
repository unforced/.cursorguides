# Project Structure Template

**Last Updated**: March 19, 2024  
**Related Guides**: [Project Setup](../00-project-setup.md)  
**Prerequisite Knowledge**: React, Vite, TypeScript

## Overview

This template provides a standardized project structure for the Bump application. Following this structure ensures consistent organization, improves developer experience, and makes the codebase more maintainable.

## Directory Structure

```
bump/
├── .cursorguides/        # Cursor AI guides for development
├── .cursorrules          # Rules for Cursor AI
├── .cursortrack.md       # Project tracking document
├── .env.development      # Development environment variables
├── .env.production       # Production environment variables
├── .env.test             # Test environment variables
├── .gitignore            # Git ignore file
├── index.html            # Entry HTML file
├── package.json          # Dependencies and scripts
├── README.md             # Project documentation
├── tsconfig.json         # TypeScript configuration
├── vite.config.ts        # Vite configuration
├── vitest.config.ts      # Vitest configuration
├── public/               # Static assets
│   ├── favicon.ico       # Favicon
│   ├── manifest.json     # PWA manifest
│   ├── robots.txt        # Robots file
│   └── assets/           # Other static assets
│       ├── icons/        # App icons
│       └── images/       # Static images
├── src/                  # Source code
│   ├── App.tsx           # Main App component
│   ├── main.tsx          # Entry point
│   ├── vite-env.d.ts     # Vite type declarations
│   ├── components/       # Reusable components
│   │   ├── common/       # Shared components
│   │   │   ├── Button/   # Button component
│   │   │   │   ├── Button.tsx
│   │   │   │   ├── Button.test.tsx
│   │   │   │   └── index.ts
│   │   │   ├── Card/     # Card component
│   │   │   ├── Input/    # Input component
│   │   │   └── ...
│   │   ├── layout/       # Layout components
│   │   │   ├── Header/   # Header component
│   │   │   ├── Footer/   # Footer component
│   │   │   ├── Sidebar/  # Sidebar component
│   │   │   └── ...
│   │   └── features/     # Feature-specific components
│   │       ├── Auth/     # Auth components
│   │       ├── Places/   # Places components
│   │       ├── Status/   # Status components
│   │       └── ...
│   ├── hooks/            # Custom hooks
│   │   ├── useAuth.ts    # Authentication hook
│   │   ├── useForm.ts    # Form handling hook
│   │   ├── useProfile.ts # Profile data hook
│   │   └── ...
│   ├── pages/            # Page components
│   │   ├── AuthPage/     # Authentication page
│   │   │   ├── AuthPage.tsx
│   │   │   ├── AuthPage.test.tsx
│   │   │   └── index.ts
│   │   ├── HomePage/     # Home page
│   │   ├── PlacesPage/   # Places page
│   │   ├── FriendsPage/  # Friends page
│   │   ├── MeetupsPage/  # Meetups page
│   │   ├── SettingsPage/ # Settings page
│   │   └── ...
│   ├── routes/           # Routing
│   │   ├── AppRoutes.tsx # Main routes
│   │   ├── PrivateRoute.tsx # Protected route
│   │   └── ...
│   ├── services/         # API services
│   │   ├── authService.ts # Auth API
│   │   ├── profileService.ts # Profile API
│   │   ├── placeService.ts # Places API
│   │   └── ...
│   ├── store/            # State management
│   │   ├── AuthContext.tsx # Auth context
│   │   ├── ThemeContext.tsx # Theme context
│   │   └── ...
│   ├── styles/           # Global styles
│   │   ├── globalStyles.ts # Global styles
│   │   ├── theme.ts     # Theme configuration
│   │   └── ...
│   ├── types/            # TypeScript types
│   │   ├── supabase.ts  # Supabase types
│   │   ├── models.ts    # Data model types
│   │   └── ...
│   └── utils/            # Utility functions
│       ├── supabaseClient.ts # Supabase client
│       ├── formatters.ts # Formatting utilities
│       ├── validators.ts # Validation utilities
│       └── ...
└── test/                 # Test setup and utilities
    ├── setup.ts          # Test setup
    ├── mocks/            # Test mocks
    │   ├── handlers.ts   # MSW handlers
    │   ├── server.ts     # MSW server
    │   └── ...
    └── utils/            # Test utilities
        ├── renderWithProviders.tsx # Test renderer
        └── ...
```

## File Organization Patterns

### Component Structure

Each component should be organized in its own directory with the following structure:

```
ComponentName/
├── ComponentName.tsx     # Component implementation
├── ComponentName.test.tsx # Component tests
├── ComponentName.module.css # Component styles (if using CSS modules)
└── index.ts              # Export file
```

Example `index.ts`:

```ts
export { default } from './ComponentName';
export * from './ComponentName';
```

### Page Structure

Pages follow a similar structure to components:

```
PageName/
├── PageName.tsx          # Page implementation
├── PageName.test.tsx     # Page tests
├── components/           # Page-specific components
│   ├── PageSpecificComponent.tsx
│   └── ...
└── index.ts              # Export file
```

### Hook Structure

Custom hooks should be organized in the `hooks` directory:

```
hooks/
├── useHookName.ts        # Hook implementation
├── useHookName.test.ts   # Hook tests
└── ...
```

### Service Structure

API services should be organized in the `services` directory:

```
services/
├── serviceNameService.ts # Service implementation
├── serviceNameService.test.ts # Service tests
└── ...
```

## Naming Conventions

1. **Files and Directories**
   - Use PascalCase for components: `Button.tsx`, `HomePage.tsx`
   - Use camelCase for utilities, hooks, and services: `formatDate.ts`, `useAuth.ts`
   - Use kebab-case for configuration files: `vite-config.ts`

2. **Components**
   - Use PascalCase for component names: `Button`, `ProfileCard`
   - Use descriptive names that indicate purpose: `UserAvatar`, `PlacesList`

3. **Hooks**
   - Prefix with `use`: `useAuth`, `useProfile`, `useForm`
   - Use camelCase: `useLocalStorage`, `useDarkMode`

4. **Services**
   - Suffix with `Service`: `authService`, `profileService`
   - Use camelCase: `placeService`, `statusService`

5. **Utilities**
   - Use descriptive function names: `formatDate`, `validateEmail`
   - Group related utilities in files: `formatters.ts`, `validators.ts`

## Import Order

Maintain a consistent import order in files:

1. External libraries (React, styled-components, etc.)
2. Internal modules (components, hooks, etc.)
3. Types and interfaces
4. Assets (images, styles, etc.)

Example:

```tsx
// 1. External libraries
import React, { useState, useEffect } from 'react';
import { useNavigate } from 'react-router-dom';
import styled from 'styled-components';

// 2. Internal modules
import Button from '../components/common/Button';
import { useAuth } from '../hooks/useAuth';
import { getProfile } from '../services/profileService';

// 3. Types and interfaces
import type { Profile } from '../types/models';

// 4. Assets
import { theme } from '../styles/theme';
import userIcon from '../assets/icons/user.svg';
```

## Best Practices

1. **Component Organization**
   - Keep components focused on a single responsibility
   - Extract reusable logic to custom hooks
   - Group related components in feature directories
   - Use index files for clean imports

2. **State Management**
   - Use React Context for global state
   - Keep state as close to where it's used as possible
   - Use custom hooks to encapsulate state logic
   - Consider performance implications of context updates

3. **Routing**
   - Organize routes in a central location
   - Use lazy loading for route components
   - Implement route guards for protected routes
   - Keep route paths in constants

4. **Styling**
   - Use styled-components for component styling
   - Maintain a consistent theme
   - Use CSS variables for dynamic theming
   - Keep global styles minimal

5. **Testing**
   - Co-locate tests with implementation files
   - Use descriptive test names
   - Test component behavior, not implementation details
   - Mock external dependencies

## Project Structure Checklist

Before considering your project structure complete, ensure:

- [ ] Directory structure follows the template
- [ ] Components are organized by feature and reusability
- [ ] Naming conventions are consistent
- [ ] Import order is maintained
- [ ] Files are co-located with their tests
- [ ] Global state is properly organized
- [ ] Routing is centralized and protected where needed
- [ ] Styling follows a consistent pattern
- [ ] Configuration files are properly set up

## Common Patterns

### Feature-First Organization

For larger applications, consider organizing by feature instead of type:

```
src/
├── features/
│   ├── auth/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/
│   │   ├── types/
│   │   └── utils/
│   ├── places/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/
│   │   ├── types/
│   │   └── utils/
│   └── ...
├── shared/
│   ├── components/
│   ├── hooks/
│   ├── services/
│   ├── types/
│   └── utils/
└── ...
```

### Barrel Files

Use barrel files (index.ts) to simplify imports:

```ts
// src/components/common/index.ts
export { default as Button } from './Button';
export { default as Card } from './Card';
export { default as Input } from './Input';

// Usage in other files
import { Button, Card, Input } from '../components/common';
```

### Lazy Loading

Implement lazy loading for routes to improve performance:

```tsx
// src/routes/AppRoutes.tsx
import React, { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';
import LoadingSpinner from '../components/common/LoadingSpinner';

const HomePage = lazy(() => import('../pages/HomePage'));
const PlacesPage = lazy(() => import('../pages/PlacesPage'));
const FriendsPage = lazy(() => import('../pages/FriendsPage'));
// ...

export default function AppRoutes() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Routes>
        <Route path="/" element={<HomePage />} />
        <Route path="/places" element={<PlacesPage />} />
        <Route path="/friends" element={<FriendsPage />} />
        {/* ... */}
      </Routes>
    </Suspense>
  );
}
```

## Related Patterns

- [Component Template](./component-template.md) - For creating components
- [Hook Template](./hook-template.md) - For creating custom hooks
- [Test Template](./test-template.md) - For writing comprehensive tests 