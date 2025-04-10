# Frontend Architecture & Development Guidelines

This document should be used by AI while writing frontend code for our application.

## 🔧 Architecture Overview

We follows a component-based architecture using React with TypeScript, emphasizing modularity, type safety, and maintainable code structure. The application uses **Zustand** for state management, **Ant Design** for UI components.

## 🧱 Tech Stack Summary

- **Framework:** React
- **UI Library:** Ant Design
- **State Management:** Zustand
- **Language:** TypeScript
- **Build Tool:** Create React App (via Craco for customization)
- **HTTP Client:** Axios
- **CI/CD:** GitHub Actions

## 🧠 State Management (Zustand)

All API logic and application state should be encapsulated inside **Zustand stores**.

### Store Guidelines

Stores should:

- Encapsulate API calls and response shapes
- Handle error/loading states
- Derive computed properties for components
- Manage state persistence where needed

### Store items

```typescript
// types.ts
export interface UserState {
  userData: UserData | null;
  loading: boolean;
  error: Error | null;
  fetchUserData: () => Promise<void>;
}

// userStore.ts
import { create } from 'zustand';
import { UserState } from './types';

export const useUserStore = create<UserState>((set) => ({
  userData: null,
  loading: false,
  error: null,
  fetchUserData: async () => {
    set({ loading: true });
    try {
      const data = await userService.fetchUser();
      set({ userData: data, error: null });
    } catch (error) {
      set({ error });
    } finally {
      set({ loading: false });
    }
  },
}));
```

### Best Practices

- Separate domain-specific logic into modular stores
- Use selectors and derived state via computed helpers
- Keep types and the store in two separate files—typically `featureStore.ts` for logic and `types.ts` for type definitions. This promotes clarity and reusability across the codebase.
- Avoid unnecessary state duplication between stores

## 🧩 Component Design

### Pages vs Components

- **Pages** are responsible for:
  - Composing feature components
  - Handling routing and layout
  - Initiating data fetching from stores

- **Components** are:
  - Pure and presentational
  - Reusable building blocks within or across features


### Component Guidelines

- Use **pure and presentational** components that receive props
- Avoid local state unless it is purely UI-related
- Use TypeScript interfaces to define prop types
- Component composition should go from atomic to compound
- Page components are responsible for fetching data and using stores

### Component Structure

```typescript
interface ButtonProps {
  label: string;
  onClick: () => void;
  type?: 'primary' | 'default';
}

export const Button: React.FC<ButtonProps> = ({ label, onClick, type = 'primary' }) => {
  return <Button type={type} onClick={onClick}>{label}</Button>;
};
```

## 📁 Project Structure

### Folder Naming Guidelines
- Use `camelCase` for folders and filenames (e.g. `userSettings`)
- Use `PascalCase` for React components (e.g. `UserSettingsCard.tsx`)
- Use `featureStore.ts` for each store logic
- Use `types.ts` for all type definitions

```
/src
├── assets/             # Static files, images, icons
├── components/         # Reusable components
├── pages/              # Page-level components that use internal feature components
├── store/              # Zustand stores
│   └── authStore       # Auth store files including types and authStore
├── services/           # API services
├── hooks/              # Custom React hooks
├── utils/              # Utility functions
├── routes/             # Route definitions
└── models/             # TypeScript types/interfaces
```

## ✅ DOs and ❌ DON'Ts

### ✅ DO

- Encapsulate all state logic in Zustand stores
- Create reusable, testable, and pure components
- Use TypeScript strictly and define all API response types
- Use Ant Design UI patterns and components
- Use consistent file naming and folder structure

### ❌ DON'T

- Don't mutate state directly outside of store functions
- Don't use `any` unless absolutely necessary
- Don't access store state directly in deeply nested components (use hooks/selectors)
- Don't write complex logic in UI components
- Don't use inline styling
- Don't commit commented code or unused imports

## 🪄 Styling & Design System

### Guidelines

- Use **Ant Design components** directly
- Use **Ant Design Grid system** for layout

### Responsive Example

```tsx
<Row gutter={16}>
  <Col xs={24} lg={12}><ComponentA /></Col>
  <Col xs={24} lg={12}><ComponentB /></Col>
</Row>
```

## 🔄 Routing

### Example Lazy Loading with Route Constants
```tsx
// routes/index.tsx
import { lazy } from 'react';

export const ROUTES = {
  HOME: '/',
  SETTINGS: '/settings',
} as const;

const HomePage = lazy(() => import('../pages/home'));
```


### Route Configuration

```tsx
// routes/index.ts
export const ROUTES = {
  HOME: '/',
  SETTINGS: '/settings',
} as const;
```

- Use enums/constants for all route paths
- Implement route guards where needed
- Co-locate route components with corresponding pages
- Lazy load large route modules

## 🧪 Testing Guidelines

### Mocking Axios with axios-mock-adapter

We use [`axios-mock-adapter`](https://github.com/ctimmerm/axios-mock-adapter) to mock HTTP requests during tests.

```ts
// __tests__/userService.test.ts
import axios from 'axios';
import MockAdapter from 'axios-mock-adapter';
import { fetchUser } from '../services/userService';

const mock = new MockAdapter(axios);

describe('fetchUser', () => {
  afterEach(() => mock.reset());

  it('should return user data when API succeeds', async () => {
    const mockUser = { id: 1, name: 'Alice' };
    mock.onGet('/api/user').reply(200, mockUser);

    const result = await fetchUser();
    expect(result).toEqual(mockUser);
  });

  it('should throw error on failure', async () => {
    mock.onGet('/api/user').reply(500);

    await expect(fetchUser()).rejects.toThrow();
  });
});
```


## 🔁 Custom Hooks Guidelines
- Place custom hooks under `/hooks`
- Prefix with `use`, e.g., `useAuthRedirect.ts`
- Use hooks to encapsulate reusable logic across pages/components

### Practices

- Use **React Testing Library** and `@testing-library/jest-dom`
- Focus on user behavior and accessibility
- Mock Zustand stores using wrapper utils
- Avoid testing internal implementation

```tsx
describe('Button', () => {
  it('calls onClick when clicked', () => {
    const onClick = jest.fn();
    render(<Button label="Submit" onClick={onClick} />);
    fireEvent.click(screen.getByText('Submit'));
    expect(onClick).toHaveBeenCalled();
  });
});
```

## 🔐 API & Error Handling

### API Pattern

```ts
export async function fetchUser(): Promise<User> {
  try {
    const res = await axios.get<User>('/api/user');
    return res.data;
  } catch (err) {
    throw new Error(getApiErrorMessage(err));
  }
}
```

- Always handle API errors
- Use `getApiErrorMessage()` for consistent UX

### Environment Variables

- Use `.env` files to store environment-specific configuration
- Follow the naming convention `NAME_APP_*` for all variables (App Name as prefix)
- Never commit sensitive values to version control

Example `.env` structure:
```plaintext
NAME_APP_API_URL=https://api.example.com
NAME_APP_ENV=development
NAME_APP_VERSION=$npm_package_version
```

Usage in code:
```typescript
const apiUrl = import.meta.env.NAME_APP_API_URL;
const environment = import.meta.env.NAME_APP_ENV;
```

Best Practices:
- Create `.env.example` with dummy values for documentation
- Add `.env` to `.gitignore`
- Use TypeScript declarations for environment variables
- Validate required env variables on app startup

## 🎯 Performance Considerations

- Use `React.memo` and `useMemo` for expensive computations
- Lazy load route-based features
- Optimize Zustand selectors to reduce re-renders
- Avoid unnecessary renders with memoization

## 📚 Documentation

- Update README and in-code comments
- Document all types and functions
- Use JSDoc or inline comments for complex logic
- Keep onboarding documentation fresh

---

This document serves as the **source of truth** for building maintainable, scalable, and performant frontend systems using React, TypeScript, Zustand, and Ant Design.&#x20;

