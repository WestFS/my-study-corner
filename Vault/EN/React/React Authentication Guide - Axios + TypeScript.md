# Robust Authentication Implementation in React: A Comprehensive Guide with Axios, Context API, and Security Best Practices
#tag: EN/React
## Introduction

Authentication is a fundamental pillar for the security and functionality of any web application, ensuring that only authorized users access specific resources. In Single Page Applications (SPAs) built with React, managing authentication state efficiently and securely is a central challenge. This report will explore how React's Context API, a native tool for global state management, can be combined with Axios, a robust and popular HTTP client, to build a scalable and secure authentication system. The focus will be an in-depth analysis of AuthContext, AuthProvider, and Axios, detailing their implementation, security best practices, and advanced patterns such as token refresh and route protection, aiming to provide a complete guide for senior developers and technical leaders.

## I. Authentication Fundamentals in React with Context API

### 1.1. What is Context API and Why Use It for Authentication?

Context API is an intrinsic React feature that allows sharing values throughout the component tree, eliminating the need to manually pass properties at each level, a problem known as "prop drilling". This API consists of `React.createContext`, which creates a Context object, and two associated components: the Provider and the Consumer. The Provider is used to wrap the portion of the component tree that needs access to the context, transmitting a value to its descendants. To consume this value in functional components, the `useContext` hook is the recommended approach in modern React.

For authentication, Context API offers significant advantages. The main benefit is eliminating "prop drilling", which simplifies component structure and improves code readability, since authentication state is frequently needed in various parts of the application. Additionally, login and logout logic, along with authentication state management, can be centralized in the AuthProvider, promoting code reuse throughout the application. Finally, all authentication-related data is stored and shared in a single location, which simplifies management and state tracking, ensuring consistency and reducing potential errors.

It's important to recognize that while `useContext` is effective for "complex application-wide state management" and for "smaller reusable multi-component APIs", it's crucial not to "abuse Context", as excessive use can diminish component reusability. Context API is better suited for "small to medium applications with little need for global state management" or for "basic data passing requirements", such as authentication. For "large and complex applications", solutions like Redux are often preferable. This indicates that while Context API is a powerful tool for authentication, it doesn't position itself as a universal solution for all complex global states, but rather as a targeted tool that excels in specific scenarios, such as authentication management.

The following table summarizes the benefits of Context API specifically for authentication management:

**Table 1: Context API Benefits for Authentication**

|Benefit|Description|
|---|---|
|Eliminates Prop Drilling|Allows direct access to authentication state without manually passing props through each level of the component tree|
|Reusable Logic|Centralizes login/logout functions and state logic in a single AuthProvider, facilitating reuse|
|Simplified State Management|Consolidates all authentication-related data and functions in one place, improving consistency and reducing errors|

### 1.2. Structure of AuthContext and AuthProvider

The starting point for implementing authentication with Context API is creating the context object using `createContext()`. A recommended practice is to export a custom hook, such as `useAuth()`, which encapsulates the `useContext(AuthContext)` call. This approach simplifies context consumption in other components and centralizes access logic, making code cleaner and more modular.

```typescript
// src/context/AuthContext.tsx
import React, { createContext, useContext, useState, useEffect, ReactNode } from 'react';

interface User {
  id: string;
  email: string;
  name: string;
  // Add other user properties as needed
}

interface AuthContextType {
  user: User | null;
  loading: boolean;
  login: (credentials: LoginCredentials) => Promise<void>;
  logout: () => Promise<void>;
  signUp?: (userData: SignUpData) => Promise<void>;
}

interface LoginCredentials {
  email: string;
  password: string;
}

interface SignUpData {
  email: string;
  password: string;
  name: string;
}

const AuthContext = createContext<AuthContextType | null>(null);

export function useAuth(): AuthContextType {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
}
```

Creating a custom hook like `useAuth()` goes beyond the mere convenience of not having to import `useContext` and the `AuthContext` itself in each component. It centralizes context dependency, meaning that if the internal structure of AuthContext changes (for example, if there's a transition from `useState` to `useReducer` for state management, or if new values are added to the context), most consuming components won't need to be modified. Only the `useAuth` hook would need to be updated. This centralization significantly improves maintainability and developer experience, a crucial aspect for software architects.

The AuthProvider is a React component that wraps the part of the application that needs access to authentication state. It's responsible for managing authentication state (such as `user`, `isLoggedIn`, `loading`) and functions to manipulate this state (`login`, `logout`, `signUp`). The AuthProvider can use `useState` for simpler states or `useReducer` for more complex state logic, offering an approach similar to Redux but native to React. The component renders `AuthContext.Provider` and passes the relevant state and functions through the `value` property. The `children` property ensures that nested components are rendered and have access to the context.

```typescript
// src/context/AuthContext.tsx (continuation)
interface AuthProviderProps {
  children: ReactNode;
}

export const AuthProvider: React.FC<AuthProviderProps> = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState<boolean>(true);

  // Initialization logic (e.g., check token on load)
  useEffect(() => {
    const initializeAuth = async () => {
      try {
        const storedUser = localStorage.getItem('user');
        if (storedUser) {
          setUser(JSON.parse(storedUser) as User);
        }
      } catch (error) {
        console.error('Error initializing auth:', error);
      } finally {
        setLoading(false);
      }
    };

    initializeAuth();
  }, []);

  const login = async (credentials: LoginCredentials): Promise<void> => {
    try {
      setLoading(true);
      // Actual login logic with API
      // const response = await apiClient.post('/auth/login', credentials);
      // setUser(response.data.user);
      // localStorage.setItem('access', response.data.access);
      // localStorage.setItem('refresh', response.data.refresh);
    } catch (error) {
      console.error('Login failed:', error);
      throw error;
    } finally {
      setLoading(false);
    }
  };

  const logout = async (): Promise<void> => {
    try {
      // Actual logout logic with API
      // await apiClient.post('/auth/logout');
      localStorage.removeItem('access');
      localStorage.removeItem('refresh');
      localStorage.removeItem('user');
      setUser(null);
    } catch (error) {
      console.error('Logout failed:', error);
      throw error;
    }
  };

  const authContextValue: AuthContextType = {
    user,
    loading,
    login,
    logout,
  };

  return (
    <AuthContext.Provider value={authContextValue}>
      {!loading && children}
    </AuthContext.Provider>
  );
};
```

For the authentication context to be available throughout the application, the AuthProvider must wrap the App component (or the parts of the application that need authentication) in the `index.tsx` or `App.tsx` file.

```typescript
// src/index.tsx or App.tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import { AuthProvider } from './context/AuthContext';
import App from './App';

const root = ReactDOM.createRoot(
  document.getElementById('root') as HTMLElement
);

root.render(
  <React.StrictMode>
    <AuthProvider>
      <App />
    </AuthProvider>
  </React.StrictMode>
);
```

### 1.3. Performance Considerations with Context API

A critical aspect of Context API, especially in large-scale applications or those with frequent updates, is that when the Provider's value changes, all components that consume that context will re-render. This occurs even if the specific part of the context that a component uses hasn't been modified. The comparison between the old and new value of the Provider's `value` property is performed using the `Object.is` algorithm. This characteristic can lead to performance bottlenecks if not managed properly.

Context API performance bottlenecks, particularly unnecessary re-renders, require careful design choices for scalable applications. All consuming components of a context re-render when the Provider's value changes, regardless of which part of the value they use. This is a direct consequence of Context API's design and React's rendering model. For a senior developer, the implication is clear: while Context API simplifies data passing, it introduces a performance cost that must be mitigated through optimization techniques.

To mitigate unnecessary re-renders and optimize performance, several strategies can be employed:

#### Split Contexts

An effective approach is to segment the context into smaller, more specific units. For example, instead of a single AuthContext containing `user`, `isLoggedIn`, `login`, and `logout`, you could create an `AuthStatusContext` for login status (`isLoggedIn`) and an `AuthActionsContext` for functions (`login`/`logout`). This separation ensures that consumers only re-render when the parts of the context relevant to them actually change.

#### Memoization

**useMemo**: It's crucial to use `useMemo` to memoize the value object passed to the Provider. This ensures the object is only recreated when its underlying dependencies change. Creating objects or functions inline directly in the Provider's `value` property will cause the context to be considered "new" on every render, forcing all consuming components to re-render unnecessarily.

**useCallback**: If functions like `login` and `logout` are passed through the context, it's crucial to use `useCallback` to memoize them. This prevents functions from being recreated on every Provider render, which could otherwise trigger re-renders in child components that depend on these functions.

**React.memo**: While it doesn't prevent the context update propagation itself, applying `React.memo` to child components that consume the context can prevent them from re-rendering if their own properties haven't changed.

#### Selective Consumption

Consuming the context only where strictly necessary is a recommended practice. Components deeper in the tree might be re-rendered unnecessarily if they consume a context that rarely changes or is irrelevant to their functionality.

```typescript
// Example of optimized AuthProvider with memoization
export const AuthProvider: React.FC<AuthProviderProps> = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState<boolean>(true);

  const login = useCallback(async (credentials: LoginCredentials): Promise<void> => {
    // Login implementation
  }, []);

  const logout = useCallback(async (): Promise<void> => {
    // Logout implementation
  }, []);

  const authContextValue = useMemo((): AuthContextType => ({
    user,
    loading,
    login,
    logout,
  }), [user, loading, login, logout]);

  return (
    <AuthContext.Provider value={authContextValue}>
      {!loading && children}
    </AuthContext.Provider>
  );
};
```

It's important to note that Context API is not optimized for high-frequency update scenarios, such as animations or rapidly changing data. This is because it triggers re-renders for all consumers with each update. For these cases, it's more appropriate to manage state locally within components or use more granular state management libraries like Redux or MobX, which are designed for this type of scenario.

## II. HTTP Request Management with Axios for Authentication

### 2.1. Introduction to Axios

Axios is a widely recognized Promise-based library used for making HTTP requests in both browser and Node.js environments. Its popularity stems from its simplicity, extensible interface, and robust feature set essential for building efficient and secure authentication flows. Among these features are request and response interceptors, error handling, and the ability to cancel requests. The library significantly simplifies configuration and request setup, allowing developers to focus more on their application logic.

### 2.2. Creating Custom Axios Instances

Instead of using the global axios instance directly in each component or function, the best practice is to create a custom Axios instance using the `axios.create()` method. This approach allows defining a `baseURL` (your API's base URL) and default headers (such as `Content-Type: application/json`) in a single location, avoiding code repetition and centralizing backend communication configuration.

```typescript
// src/api/axiosInstance.ts
import axios, { AxiosInstance, AxiosError } from 'axios';

interface ApiError {
  message: string;
  status?: number;
  data?: any;
}

const apiClient: AxiosInstance = axios.create({
  baseURL: process.env.REACT_APP_API_BASE_URL || 'http://localhost:5000/api',
  timeout: 10000, // Request timeout in ms
  headers: {
    'Content-Type': 'application/json',
  },
});

export default apiClient;
```

Creating custom Axios instances is fundamental for a maintainable and scalable API layer, especially in microservices architectures. The recommendation to use `axios.create()` goes beyond a simple "best practice". It helps avoid redundancy of writing the full URL in each request and simplifies the deployment process. In environments with multiple microservices, it's possible to create separate Axios instances for each service, each with its own `baseURL` and specific interceptors. This ability to have distinct configurations and interceptors for different services is an architectural decision that directly impacts the maintainability and scalability of a complex application.

The benefits of centralizing API logic and avoiding URL hardcoding are multiple:

- **Avoids Redundancy**: No need to repeat the base URL and headers in each individual request
- **Cleaner and More Manageable Code**: API configuration is centralized, making code more readable, organized, and easier to maintain
- **Facilitates Refactoring and Deployment**: If the API URL changes (e.g., from a local development environment to a production domain), the change needs to be made in only one place
- **Organization in Microservices Architectures**: In scenarios where the application interacts with multiple microservices, dedicated Axios instances can be created for each service

### 2.3. Axios Interceptors: Centralized Request and Response Handling

Axios offers a powerful mechanism called "interceptors" that allows intervening in requests before they're sent and in responses before they're processed by `then` or `catch` blocks. This capability is fundamental for modifying requests, responses, or handling errors globally and centrally.

#### Request Interceptors: Automatic Addition of Authorization Tokens

One of the most common and important use cases for request interceptors is automatically adding authorization tokens (such as JWTs in Bearer format) to all requests' headers, or selectively for specific services. This eliminates the need to manually add the token in each API call, reducing code duplication and ensuring all authenticated requests include the necessary token.

```typescript
// src/api/axiosInstance.ts (continuation)
import { AxiosRequestConfig } from 'axios';

interface AuthTokens {
  access: string;
  refresh: string;
}

// Request interceptor for adding authorization headers
apiClient.interceptors.request.use(
  (config: AxiosRequestConfig) => {
    const accessToken = localStorage.getItem('access');
    if (accessToken && config.headers) {
      config.headers['Authorization'] = `Bearer ${accessToken}`;
    }
    return config;
  },
  (error: AxiosError) => {
    return Promise.reject(error);
  }
);
```

#### Response Interceptors: Global Error Handling and Response Modification

Response interceptors are ideal for managing common HTTP errors (such as 401 Unauthorized, 404 Not Found, 500 Internal Server Error) in a centralized manner. Through them, it's possible to display generic error messages to users, automatically redirect to the login page in case of a 401 error, or log errors for debugging and monitoring purposes. Additionally, different microservices may return data in various formats; a response interceptor can standardize these formats so the frontend always receives data consistently, simplifying consumption.

```typescript
// src/api/axiosInstance.ts (continuation)
import { AxiosResponse } from 'axios';

// Response interceptor for global error handling
apiClient.interceptors.response.use(
  (response: AxiosResponse) => response,
  (error: AxiosError) => {
    if (error.response) {
      console.error('API Error:', error.response.status, error.response.data);
      
      // Handle specific error statuses
      switch (error.response.status) {
        case 401:
          // Handle unauthorized access
          localStorage.removeItem('access');
          localStorage.removeItem('refresh');
          // Redirect to login or dispatch logout action
          break;
        case 403:
          // Handle forbidden access
          break;
        case 404:
          // Handle not found
          break;
        case 500:
          // Handle server errors
          break;
        default:
          // Handle other errors
          break;
      }
    } else if (error.request) {
      console.error('No response received:', error.request);
    } else {
      console.error('Request configuration error:', error.message);
    }
    
    return Promise.reject(error);
  }
);
```

Interceptors are the pillar for robust and transparent authentication, abstracting complexity from components. The ability of interceptors to "handle errors application-wide" and "send authorization tokens in headers on each request" is more than a convenience. Complete implementations demonstrate how interceptors automatically manage token addition and refresh flow. Centralized API logic and consistent error handling are enhanced. This reveals that interceptors allow React components to focus purely on UI logic, while the network layer autonomously manages authentication, retries, and error normalization.

**Table 2: Summary of Axios Interceptors for Authentication**

|Interceptor Type|Authentication Use Case|Benefits|
|---|---|---|
|Request Interceptor|Automatic addition of authorization headers (e.g., Bearer JWT) to all requests|Ensures all authenticated requests include the token without code duplication|
|Response Interceptor|Global error handling (e.g., 401 Unauthorized, 404 Not Found), login redirection|Centralizes error handling logic, provides consistent user feedback, and simplifies debugging|
|Response Interceptor|Token refresh logic implementation for automatic renewal of expired tokens|Keeps user session active without requiring repeated login, improves user experience and security|
|Response Interceptor|API response format modification or standardization|Ensures frontend receives data in consistent format regardless of API source|

## III. Complete Authentication Flow Implementation in React

### 3.1. Login and Logout Logic

The `logi
n` and `logout` functions are the central methods exposed by AuthContext for manipulating authentication state, implemented within the AuthProvider.

The `login` function is responsible for receiving user credentials (such as username and password) and making a call to the login API using the configured Axios instance. On success, it stores the received tokens (access and refresh tokens) and updates the `user` state in AuthContext. Typically, after successful login, the application can redirect the user to a protected route, such as a dashboard (`/dashboard`).

The `logout` function handles making a call to the logout API (if the backend requires token invalidation on the server). It then removes stored tokens (whether from localStorage or cookies) and sets the `user` state to null in AuthContext, indicating the user is no longer authenticated. Finally, the application redirects the user to the login page.

```typescript
// src/context/AuthContext.tsx (extended implementation)
import apiClient from '../api/axiosInstance';

interface AuthResponse {
  user: User;
  access: string;
  refresh: string;
}

export const AuthProvider: React.FC<AuthProviderProps> = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState<boolean>(true);

  const login = useCallback(async (credentials: LoginCredentials): Promise<void> => {
    try {
      setLoading(true);
      const response = await apiClient.post<AuthResponse>('/auth/login', credentials);
      const { user, access, refresh } = response.data;
      
      // Store tokens
      localStorage.setItem('access', access);
      localStorage.setItem('refresh', refresh);
      localStorage.setItem('user', JSON.stringify(user));
      
      setUser(user);
    } catch (error) {
      console.error('Login failed:', error);
      throw error;
    } finally {
      setLoading(false);
    }
  }, []);

  const logout = useCallback(async (): Promise<void> => {
    try {
      // Call logout API if backend requires it
      await apiClient.post('/auth/logout');
    } catch (error) {
      console.error('Logout API call failed:', error);
    } finally {
      // Clear tokens and user state regardless of API call success
      localStorage.removeItem('access');
      localStorage.removeItem('refresh');
      localStorage.removeItem('user');
      setUser(null);
    }
  }, []);

  // ... rest of the implementation
};
```

Login form components, such as `LoginComponent`, use the `useAuth()` hook to access the login and logout functions. Form field state (e.g., `email`, `password`) is managed locally with `useState`, and the `handleSubmit` function is responsible for invoking the context's login function, passing the user-entered credentials.

```typescript
// src/components/LoginComponent.tsx
import React, { useState, FormEvent } from 'react';
import { useAuth } from '../context/AuthContext';

const LoginComponent: React.FC = () => {
  const [email, setEmail] = useState<string>('');
  const [password, setPassword] = useState<string>('');
  const [error, setError] = useState<string>('');
  const [isLoading, setIsLoading] = useState<boolean>(false);
  const { login } = useAuth();

  const handleSubmit = async (e: FormEvent<HTMLFormElement>): Promise<void> => {
    e.preventDefault();
    setError('');
    setIsLoading(true);

    try {
      await login({ email, password });
      // Redirect is typically handled by AuthProvider or ProtectedRoute
    } catch (error) {
      setError('Login failed. Please check your credentials.');
      console.error('Login error:', error);
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      {error && <div className="error-message">{error}</div>}
      <div>
        <input
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          placeholder="Email"
          required
          disabled={isLoading}
        />
      </div>
      <div>
        <input
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          placeholder="Password"
          required
          disabled={isLoading}
        />
      </div>
      <button type="submit" disabled={isLoading}>
        {isLoading ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
};

export default LoginComponent;
```

### 3.2. Session Persistence and Token Management

The choice of storage location for authentication tokens is a crucial security decision that directly impacts the application's resilience to certain types of attacks.

#### localStorage vs. Cookies (HttpOnly and Secure) for Token Storage

**localStorage:**

- **XSS (Cross-Site Scripting) Vulnerability**: Tokens stored in localStorage are directly accessible via JavaScript. If the application has an XSS vulnerability, an injected malicious script can easily steal the token from localStorage, allowing an attacker to impersonate the user.
- **Not recommended for session tokens**: Due to their inherent vulnerability to XSS attacks, many experts strongly discourage using localStorage for storing authentication tokens.
- **Use in JWT Bearer Tokens (with caveats)**: While not the most secure option, if the backend uses JWT bearer tokens, localStorage can be employed for storage. However, the HTTP client needs to be explicitly configured to include the token in all subsequent requests.

**Cookies (with HttpOnly and Secure):**

- **Better Security for Web Apps**: For web applications, server-side cookie-based authentication is generally considered the most secure approach.
- **HttpOnly Flag**: This flag prevents the cookie from being accessed or manipulated via JavaScript. This significantly mitigates XSS attack risk, as even if a malicious script is injected, it cannot steal the token contained in the cookie.
- **Secure Flag**: Ensures the cookie is transmitted only over HTTPS (secure and encrypted) connections. This protects the token against interception during transmission on unsecured networks.
- **CSRF (Cross-Site Request Forgery) Vulnerability**: Cookies are automatically sent by the browser in requests to the origin domain, making them inherently vulnerable to CSRF attacks. However, this vulnerability can be mitigated with techniques like anti-CSRF tokens (e.g., "double cookie submit pattern") or applying the `SameSite=Strict` flag to cookies.

**Table 3: Token Storage Comparison (localStorage vs. Cookies)**

|Characteristic|localStorage|Cookies (HttpOnly & Secure)|
|---|---|---|
|JavaScript Accessibility|Yes|No (HttpOnly prevents)|
|XSS Vulnerability|High|Low (protected by HttpOnly)|
|Automatic Sending in Requests|No (requires manual addition)|Yes (automatic by browser)|
|CSRF Vulnerability|Low (if tokens not exposed)|High (without mitigation, mitigable with SameSite/anti-CSRF)|
|Persistence|Yes (persists until removed)|Yes (controlled by expires or max-age)|
|General Recommendation for Auth Tokens|Avoid|Preferable (with HttpOnly and Secure)|

#### Refresh Token Implementation with Axios Interceptors for Automatic Token Renewal

Access tokens typically have a short lifespan for security reasons. When these tokens expire, subsequent API requests result in a 401 (Unauthorized) error. To avoid requiring users to log in again with each token expiration, a refresh token is used, which has a longer lifespan and is specifically employed to obtain a new access token.

The refresh token logic is ideally implemented in an Axios response interceptor, following these steps:

1. **Capture 401 Error**: The response interceptor is configured to capture errors with HTTP status 401
2. **Retry Verification**: A flag, such as `_retry`, is added to the original request configuration to avoid infinite retry loops
3. **Refresh Request**: A new request is made to the token refresh endpoint (e.g., `/refresh`), sending the stored refresh token
4. **Token Update**: If the refresh request succeeds, the new access token is stored and the original request's Authorization header is updated
5. **Original Request Retry**: The original request that failed with 401 is automatically retried with the new access token
6. **Refresh Failure Handling**: If the refresh token is invalid or expired, the user is logged out and redirected to the login page

```typescript
// src/api/axiosInstance.ts (extended with refresh token logic)
import axios, { AxiosResponse, AxiosError, AxiosRequestConfig } from 'axios';

interface RefreshTokenResponse {
  access: string;
  refresh?: string;
}

let isRefreshing = false;
let failedQueue: Array<{
  resolve: (value: string) => void;
  reject: (error: any) => void;
}> = [];

const processQueue = (error: AxiosError | null, token: string | null = null) => {
  failedQueue.forEach(promise => {
    if (error) {
      promise.reject(error);
    } else {
      promise.resolve(token!);
    }
  });
  
  failedQueue = [];
};

// Enhanced response interceptor with refresh token logic
apiClient.interceptors.response.use(
  (response: AxiosResponse) => response,
  async (error: AxiosError) => {
    const originalRequest = error.config as AxiosRequestConfig & { _retry?: boolean };

    if (error.response?.status === 401 && !originalRequest._retry) {
      if (isRefreshing) {
        return new Promise((resolve, reject) => {
          failedQueue.push({ resolve, reject });
        }).then((token: string) => {
          if (originalRequest.headers) {
            originalRequest.headers['Authorization'] = `Bearer ${token}`;
          }
          return apiClient(originalRequest);
        }).catch(err => {
          return Promise.reject(err);
        });
      }

      originalRequest._retry = true;
      isRefreshing = true;

      try {
        const refreshToken = localStorage.getItem('refresh');
        if (!refreshToken) {
          throw new Error('No refresh token available');
        }

        const response = await axios.post<RefreshTokenResponse>('/api/auth/refresh', {
          refresh: refreshToken
        });

        const { access, refresh: newRefreshToken } = response.data;
        
        localStorage.setItem('access', access);
        if (newRefreshToken) {
          localStorage.setItem('refresh', newRefreshToken);
        }

        // Update default header for future requests
        if (apiClient.defaults.headers) {
          apiClient.defaults.headers.common['Authorization'] = `Bearer ${access}`;
        }

        processQueue(null, access);
        
        if (originalRequest.headers) {
          originalRequest.headers['Authorization'] = `Bearer ${access}`;
        }
        
        return apiClient(originalRequest);
      } catch (refreshError) {
        processQueue(refreshError as AxiosError, null);
        
        // Clear tokens and redirect to login
        localStorage.removeItem('access');
        localStorage.removeItem('refresh');
        localStorage.removeItem('user');
        
        // Redirect to login page
        window.location.href = '/login';
        
        return Promise.reject(refreshError);
      } finally {
        isRefreshing = false;
      }
    }

    return Promise.reject(error);
  }
);

export default apiClient;
```

#### Strategies for Handling Concurrent Requests During Token Refresh

A common challenge in refresh token implementation occurs when multiple requests are made simultaneously with an expired access token. This can lead to multiple refresh token calls to the server, which may overload infrastructure or, worse, cause the server to invalidate the refresh token after the first successful use, resulting in failures for other refresh requests and forcing unexpected user logout.

To mitigate this problem, it's essential to implement a request queue or "single refresh call" mechanism. The approach consists of, upon detecting the first request that fails with a 401 error, initiating the refresh token function. All subsequent requests that also fail with 401 while the refresh process is ongoing are paused and added to a queue. Once the new access token is obtained, all requests in the queue are retried with the new valid token.

### 3.3. Protected Routes (ProtectedRoute) with react-router-dom

Protected routes are essential components that control access to certain parts of the application based on the user's authentication status. In React applications, this functionality is commonly implemented in conjunction with react-router-dom and AuthContext.

A ProtectedRoute component checks the user's authentication state (for example, the presence of a `user` or the value of `isLoggedIn`) from AuthContext. If the user is authenticated, the component renders the protected route content, typically through the `Outlet` component (in react-router-dom v6+) or the route's element. If the user is not authenticated, the ProtectedRoute component automatically redirects them to the login page. Programmatic redirection is performed using the `useNavigate` hook or the `Navigate` component from react-router-dom.

```typescript
// src/components/ProtectedRoute.tsx
import React from 'react';
import { Navigate, Outlet } from 'react-router-dom';
import { useAuth } from '../context/AuthContext';

interface ProtectedRouteProps {
  redirectTo?: string;
}

const ProtectedRoute: React.FC<ProtectedRouteProps> = ({ redirectTo = '/login' }) => {
  const { user, loading } = useAuth();

  if (loading) {
    return (
      <div className="loading-container">
        <div>Loading authentication...</div>
        {/* Or a loading spinner component */}
      </div>
    );
  }

  return user ? <Outlet /> : <Navigate to={redirectTo} replace />;
};

export default ProtectedRoute;
```

It's crucial to understand that client-side protected routes are a user experience (UX) feature, not an autonomous security mechanism; backend validation is paramount. Protected routes implementation is not a way to protect

# React Authentication Guide - TypeScript Edition

## 3.3. Protected Routes (ProtectedRoute) with react-router-dom

Protected routes are essential components that control access to certain parts of the application based on the user's authentication status. In React applications, this functionality is commonly implemented in conjunction with react-router-dom and AuthContext.

A ProtectedRoute component checks the user's authentication state (for example, the presence of a user or the value of isLoggedIn) from the AuthContext. If the user is authenticated, the component renders the protected route content, typically through the Outlet component (in react-router-dom v6+) or the route's element. If the user is not authenticated, the ProtectedRoute component automatically redirects them to the login page. Programmatic redirection is performed using the useNavigate hook or the Navigate component from react-router-dom.

Here's a TypeScript code example for a ProtectedRoute:

```typescript
// src/routes/ProtectedRoute.tsx
import React from 'react';
import { Navigate, Outlet } from 'react-router-dom';
import { useAuth } from '../context/AuthContext';

const ProtectedRoute: React.FC = () => {
  const { user, loading } = useAuth(); // Assumes 'loading' indicates initial authentication status

  if (loading) {
    return <div>Loading authentication...</div>; // Or a loading spinner
  }

  return user ? <Outlet /> : <Navigate to="/login" replace />;
};

export default ProtectedRoute;
```

It's crucial to understand that client-side protected routes are a user experience (UX) feature, not an autonomous security mechanism; backend validation is paramount. As explicitly stated, "implementing protected routes is not a way to protect your application, but a way to control what the client sees". The same source emphasizes that "requests to your backend must be protected with some form of authentication". This highlights that while frontend route protection is important for usability and user guidance, it doesn't replace the need for access validation on the server. This perspective reinforces the importance of a layered security approach, where authorization is always verified on the backend, regardless of what is displayed or hidden in the client interface.

The route configuration in App.tsx with ProtectedRoute is done as follows:

```typescript
// src/App.tsx (Example with react-router-dom v6+)
import React from 'react';
import { createBrowserRouter, RouterProvider } from 'react-router-dom';
import { AuthProvider } from './context/AuthContext';
import LoginComponent from './components/LoginComponent';
import Dashboard from './components/Dashboard'; // Example protected component
import HomePage from './components/HomePage'; // Example public component
import ProtectedRoute from './routes/ProtectedRoute';

const router = createBrowserRouter([
  {
    path: "/",
    element: <HomePage />,
  },
  {
    path: "/login",
    element: <LoginComponent />,
  },
  {
    path: "/",
    element: <ProtectedRoute />,
    children: [
      {
        path: "dashboard",
        element: <Dashboard />,
      },
    ],
  },
]);

const App: React.FC = () => {
  return (
    <AuthProvider>
      <RouterProvider router={router} />
    </AuthProvider>
  );
};

export default App;
```

## IV. Best Practices and Security Considerations

### 4.1. Global and Local Error Handling

Implementing a centralized error handling mechanism is vital for any application's robustness. This can be achieved by combining Axios interceptors (discussed in Section II.3) with a dedicated error context, such as an ErrorProvider. This approach allows capturing, storing, and sharing error information throughout the application consistently, improving maintainability and problem response capability.

Errors should be structured consistently, including attributes such as a descriptive message, a unique error code, and a timestamp, to facilitate handling and debugging. Beyond technical capture, it's crucial to provide immediate and consistent visual feedback to the user, using elements like error banners, modals, or toast notifications. Messages should be clear, contextual, and avoid technical jargon to ensure the user understands the problem and necessary actions. It's also important to implement logic to reset error state after a specific period or when the user interacts with the related element, preventing error messages from persisting unnecessarily.

Here's a TypeScript example of an ErrorProvider:

```typescript
// src/context/ErrorContext.tsx
import React, { createContext, useContext, useState, ReactNode } from 'react';

interface ErrorInfo {
  message: string;
  code?: string;
  timestamp: Date;
}

interface ErrorContextType {
  error: ErrorInfo | null;
  setError: (error: ErrorInfo | null) => void;
  clearError: () => void;
}

const ErrorContext = createContext<ErrorContextType | undefined>(undefined);

interface ErrorProviderProps {
  children: ReactNode;
}

export const ErrorProvider: React.FC<ErrorProviderProps> = ({ children }) => {
  const [error, setError] = useState<ErrorInfo | null>(null);

  const clearError = () => setError(null);

  return (
    <ErrorContext.Provider value={{ error, setError, clearError }}>
      {children}
    </ErrorContext.Provider>
  );
};

export const useError = (): ErrorContextType => {
  const context = useContext(ErrorContext);
  if (context === undefined) {
    throw new Error('useError must be used within an ErrorProvider');
  }
  return context;
};
```

Comprehensive error handling goes beyond technical capture, encompassing user experience and operational monitoring. This includes not only how to capture errors (with an ErrorProvider and hooks), but also the importance of "consistent user feedback" (banners, modals, toasts), "contextual messages" and "error logging and tracking" with tools like Sentry or LogRocket. This demonstrates that a robust error strategy for a senior developer is not limited to network interception (Axios), but extends to how errors are communicated to the user and how they are monitored by the development team, directly impacting user satisfaction and problem resolution time.

For traceability and monitoring purposes, it's essential to integrate centralized logging solutions, such as Sentry or LogRocket, to capture and monitor application failures in real-time. This integration improves problem traceability and can significantly reduce resolution time. Recording relevant information, such as error type, user context, and timestamp, is essential for effective analysis.

### 4.2. JWT Token Security and Attack Prevention

JWT token security is multifaceted, requiring vigilance against implementation flaws, careful payload data design, and robust revocation strategies.

**Secure Token Storage (Reinforcement):** The critical importance of using HttpOnly and Secure cookies for storing authentication tokens is reiterated, avoiding localStorage due to its inherent vulnerability to Cross-Site Scripting (XSS) attacks.

**XSS (Cross-Site Scripting) and CSRF (Cross-Site Request Forgery) Mitigation:**

- **XSS**: Beyond the protection offered by HttpOnly cookies, rigorous sanitization of all user inputs is crucial. Frameworks like React, which escape content by default (unless dangerouslySetInnerHTML is explicitly used), provide an additional layer of defense.
- **CSRF**: For cookies, it's imperative to implement anti-CSRF tokens (for example, the "double cookie submit" pattern) or use the SameSite=Strict flag on cookies. This flag instructs the browser to send cookies only for requests that originate from the same site that set them, protecting against forged requests from other domains.

**Token Validation and Revocation Management:** A JWT is considered valid while it hasn't expired and possesses a valid cryptographic signature. However, it may be necessary to revoke a token before its natural expiration (for example, in case of account compromise, forced logout, or password change).

Strategies for token revocation include:

- **Short Expiration Period + Refresh Token**: The access token has an intentionally short lifespan, while a longer-lived refresh token is used to obtain new access tokens. If the access token is compromised, the exposure time is limited. If the refresh token is stolen, its revocation is more critical, but less frequent.
- **Deny List (Blacklist)**: Maintain a list of revoked JWT tokens in external storage (like a database or cache). Each request with a JWT is checked against this list to ensure the token hasn't been invalidated.
- **Signing Key Change**: Force invalidation of all previously issued tokens, which can be a drastic but effective measure in generalized compromise scenarios.

Here's a TypeScript example of a JWT service with security considerations:

```typescript
// src/services/jwtService.ts
import axios, { AxiosResponse } from 'axios';

interface TokenResponse {
  accessToken: string;
  refreshToken: string;
  expiresIn: number;
}

interface DecodedToken {
  exp: number;
  sub: string;
  iat: number;
  // Add other expected JWT claims
}

class JWTService {
  private static instance: JWTService;
  private accessToken: string | null = null;
  private refreshToken: string | null = null;
  private tokenExpiration: number | null = null;

  private constructor() {}

  static getInstance(): JWTService {
    if (!JWTService.instance) {
      JWTService.instance = new JWTService();
    }
    return JWTService.instance;
  }

  setTokens(tokens: TokenResponse): void {
    this.accessToken = tokens.accessToken;
    this.refreshToken = tokens.refreshToken;
    this.tokenExpiration = Date.now() + tokens.expiresIn * 1000;
  }

  getAccessToken(): string | null {
    return this.accessToken;
  }

  isTokenExpired(): boolean {
    if (!this.tokenExpiration) return true;
    return Date.now() >= this.tokenExpiration;
  }

  async refreshAccessToken(): Promise<string | null> {
    if (!this.refreshToken) return null;

    try {
      const response: AxiosResponse<TokenResponse> = await axios.post('/api/auth/refresh', {
        refreshToken: this.refreshToken,
      });

      this.setTokens(response.data);
      return this.accessToken;
    } catch (error) {
      this.clearTokens();
      return null;
    }
  }

  clearTokens(): void {
    this.accessToken = null;
    this.refreshToken = null;
    this.tokenExpiration = null;
  }

  // Decode JWT payload (client-side validation only - never trust for security)
  private decodeToken(token: string): DecodedToken | null {
    try {
      const payload = token.split('.')[1];
      const decoded = JSON.parse(atob(payload));
      return decoded as DecodedToken;
    } catch (error) {
      return null;
    }
  }
}

export default JWTService;
```

**Avoiding Common JWT Pitfalls:**

- **Don't Trust the alg Header**: The alg: none vulnerability (or variations like alg: nOnE) allows an attacker to create arbitrary tokens that are accepted as valid. It's fundamental that the server always uses a static and explicit algorithm configuration to verify the token signature, ignoring the alg header value provided in the token.
- **Don't Put Sensitive Data in JWT**: The payload of a JWT is only Base64 encoded, not encrypted. Any sensitive information inserted into the JWT can be easily read by anyone who has access to the token. If sensitive data is strictly necessary, the JWT should be encrypted, which adds complexity to the system.
- **Strong and Non-Hardcoded Secrets**: Secret keys used to sign JWTs should be robust, securely generated, and never hardcoded in the application source code. They should be provided through environment variables or secret managers, ensuring they're not exposed in code repositories.
- **Comprehensive Testing**: It's essential to have a robust test suite that includes malformed, expired, or incorrectly algorithmic JWTs to ensure server validation works correctly and the application reacts securely to invalid tokens.

JWT security is multifaceted, requiring vigilance against implementation flaws (like the alg: none attack), careful payload data design, and robust revocation strategies. This goes beyond simple storage, including specific JWT vulnerabilities like the alg: none attack, the danger of sensitive data in payload, and the need for revocation mechanisms (deny list, short expiration with refresh tokens). For a senior developer, this is crucial, as it demonstrates deep understanding of JWT-specific attack vectors and their mitigations. It's not enough to just "use JWTs", but rather "use JWTs securely", which implies detailed knowledge of their weaknesses and implementation best practices.

### 4.3. Additional Design Patterns

Beyond the already mentioned practices, adopting specific design patterns can enhance the architecture and maintainability of a React authentication system:

**Centralization of API Logic in a Dedicated Service:** It's good practice to encapsulate all API calls in a dedicated service or module (for example, apiService.ts). This approach promotes separation of concerns, isolating backend communication logic from user interface components. As a result, components become cleaner and focused on their main responsibility: rendering and user interaction.

Here's a TypeScript example of a centralized API service:

```typescript
// src/services/apiService.ts
import axios, { AxiosInstance, AxiosResponse, AxiosError } from 'axios';
import JWTService from './jwtService';

interface ApiResponse<T> {
  data: T;
  message?: string;
  status: number;
}

interface LoginCredentials {
  email: string;
  password: string;
}

interface User {
  id: string;
  email: string;
  name: string;
  role: string;
}

class ApiService {
  private static instance: ApiService;
  private axiosInstance: AxiosInstance;
  private jwtService: JWTService;

  private constructor() {
    this.jwtService = JWTService.getInstance();
    this.axiosInstance = axios.create({
      baseURL: process.env.REACT_APP_API_BASE_URL || 'http://localhost:3001/api',
      timeout: 10000,
      headers: {
        'Content-Type': 'application/json',
      },
    });

    this.setupInterceptors();
  }

  static getInstance(): ApiService {
    if (!ApiService.instance) {
      ApiService.instance = new ApiService();
    }
    return ApiService.instance;
  }

  private setupInterceptors(): void {
    // Request interceptor to add auth token
    this.axiosInstance.interceptors.request.use(
      (config) => {
        const token = this.jwtService.getAccessToken();
        if (token && !this.jwtService.isTokenExpired()) {
          config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
      },
      (error: AxiosError) => Promise.reject(error)
    );

    // Response interceptor to handle token refresh
    this.axiosInstance.interceptors.response.use(
      (response: AxiosResponse) => response,
      async (error: AxiosError) => {
        const originalRequest = error.config;

        if (error.response?.status === 401 && originalRequest && !originalRequest._retry) {
          originalRequest._retry = true;

          const newToken = await this.jwtService.refreshAccessToken();
          if (newToken) {
            originalRequest.headers.Authorization = `Bearer ${newToken}`;
            return this.axiosInstance(originalRequest);
          }
        }

        return Promise.reject(error);
      }
    );
  }

  // Authentication methods
  async login(credentials: LoginCredentials): Promise<ApiResponse<{ user: User; tokens: any }>> {
    const response = await this.axiosInstance.post<ApiResponse<{ user: User; tokens: any }>>(
      '/auth/login',
      credentials
    );
    return response.data;
  }

  async logout(): Promise<void> {
    await this.axiosInstance.post('/auth/logout');
    this.jwtService.clearTokens();
  }

  async getCurrentUser(): Promise<ApiResponse<User>> {
    const response = await this.axiosInstance.get<ApiResponse<User>>('/auth/me');
    return response.data;
  }

  // Generic methods for other API calls
  async get<T>(endpoint: string): Promise<ApiResponse<T>> {
    const response = await this.axiosInstance.get<ApiResponse<T>>(endpoint);
    return response.data;
  }

  async post<T>(endpoint: string, data: any): Promise<ApiResponse<T>> {
    const response = await this.axiosInstance.post<ApiResponse<T>>(endpoint, data);
    return response.data;
  }

  async put<T>(endpoint: string, data: any): Promise<ApiResponse<T>> {
    const response = await this.axiosInstance.put<ApiResponse<T>>(endpoint, data);
    return response.data;
  }

  async delete<T>(endpoint: string): Promise<ApiResponse<T>> {
    const response = await this.axiosInstance.delete<ApiResponse<T>>(endpoint);
    return response.data;
  }
}

export default ApiService;
```

This centralization also makes API calls easily reusable across different parts of the application and simplifies maintenance. Any changes in how calls are made (such as adding a new header, modifying an endpoint, or implementing new error handling logic) can be performed in a single location, minimizing the risk of inconsistencies and bugs.

## Conclusion

Implementing a robust authentication system in React is a complex task that requires a holistic approach and the integration of various technological and security layers.

React's Context API emerges as an elegant and native solution for global authentication state management, eliminating "prop drilling" and centralizing authentication logic. However, its use requires careful optimization to avoid unnecessary re-renders in larger-scale applications, which can be mitigated with context division and memoization techniques.

Axios, with its custom instances and powerful interceptors, proves to be the ideal tool for managing HTTP requests. It allows automatic addition of authorization tokens, global error handling, and sophisticated implementation of token refresh flows, including complex management of concurrent requests. This ability to abstract network logic from UI components is fundamental for maintainability.

Security is a paramount and non-negotiable aspect. Token storage choice is critical, with clear preference for HttpOnly and Secure cookies over localStorage due to its XSS vulnerability. Mitigating attacks like XSS and CSRF, rigorous token validation, and implementing revocation strategies are essential for protecting the application. Additionally, understanding JWT-specific pitfalls, such as alg header manipulation and sensitive data exposure, is crucial for avoiding vulnerabilities.

A truly robust authentication system is a layered defense, combining client-side user experience with server-side security and continuous monitoring. This comprehensive approach demonstrates that authentication is not an isolated feature, but a web of interconnected components. The journey begins with client-side state management (Context API), advances to network communication (Axios), deepens into security concerns (token storage, JWT flaws, XSS/CSRF), and culminates with error handling and monitoring. No isolated solution is sufficient; security and robustness require a multifaceted and continuous approach, where each layer complements the others.

### Final Recommendations for Development and Maintenance:

1. **Prioritize Security**: Integrate security considerations from the early design and development phases, rather than treating them as an afterthought.
2. **Invest in Automated Testing**: Develop comprehensive tests, especially for authentication flows, token refresh, and error scenarios, to ensure functionality and security.
3. **Stay Updated**: Keep up with security best practices and emerging vulnerabilities in the authentication and web development ecosystem.
4. **Proactive Monitoring**: Implement logging and monitoring tools to quickly identify and respond to errors and suspicious activities in the application.
5. **Document Architectural Decisions**: Record design choices, especially those related to security and token management, to facilitate future understanding and maintenance.

---
## References

1. Step-by-Step Guide to React's Context API - CRS Info Solutions. Retrieved July 12, 2025, from [https://www.crsinfosolutions.com/reactjs-context-api/](https://www.crsinfosolutions.com/reactjs-context-api/)
2. useContext – React. Retrieved July 12, 2025, from [https://react.dev/reference/react/useContext](https://react.dev/reference/react/useContext)
3. Context - React. Retrieved July 12, 2025, from [https://legacy.reactjs.org/docs/context.html](https://legacy.reactjs.org/docs/context.html)
4. Predictable React authentication with the Context API - Blog - Finiam. Retrieved July 12, 2025, from [https://blog.finiam.com/blog/predictable-react-authentication-with-the-context-api](https://blog.finiam.com/blog/predictable-react-authentication-with-the-context-api)
5. Creating an authentication context with useContext in React | Jose Carrillo. Retrieved July 12, 2025, from [https://www.josecarrillo.me/creating-an-authentication-context-with-usecontext-in-react/](https://www.josecarrillo.me/creating-an-authentication-context-with-usecontext-in-react/)
6. Simplifying Authentication in React with Context API | by Sudha. Retrieved July 12, 2025, from [https://medium.com/@sudabefortune/simplifying-authentication-in-react-with-context-api-6a4b0ff9d8f1](https://medium.com/@sudabefortune/simplifying-authentication-in-react-with-context-api-6a4b0ff9d8f1)
7. Context API vs Redux: Which State Management Tool Wins? - OneClick IT Consultancy. Retrieved July 12, 2025, from [https://www.oneclickitsolution.com/blog/context-api-vs-redux](https://www.oneclickitsolution.com/blog/context-api-vs-redux)
8. Redux vs Context: Performance vs Simplicity in React State Management. Retrieved July 12, 2025, from [https://aglowiditsolutions.com/blog/react-redux-vs-context-api/](https://aglowiditsolutions.com/blog/react-redux-vs-context-api/)
9. A Complete Guide to Auth in React with Axios, JWT and Context API. Retrieved July 12, 2025, from [https://reference.nirajankhatiwada.com.np/posts/pages/react/auth/](https://reference.nirajankhatiwada.com.np/posts/pages/react/auth/)
10. Manipulating request and response with axios interceptors | by Dikshant Raj. Retrieved July 12, 2025, from [https://dikshantraj2001.medium.com/manipulating-request-and-response-with-axios-interceptors-74c3108f21cd](https://dikshantraj2001.medium.com/manipulating-request-and-response-with-axios-interceptors-74c3108f21cd)
11. Axios documentation - DevDocs. Retrieved July 12, 2025, from [https://devdocs.io/axios/](https://devdocs.io/axios/)
12. Axios API | Axios Docs. Retrieved July 12, 2025, from [https://axios-http.com/docs/api_intro](https://axios-http.com/docs/api_intro)
13. How to send Basic Auth with Axios in React & Node? - GeeksforGeeks. Retrieved July 12, 2025, from [https://www.geeksforgeeks.org/mern/how-to-send-basic-auth-with-axios-in-react-node/](https://www.geeksforgeeks.org/mern/how-to-send-basic-auth-with-axios-in-react-node/)
14. How to manage axios errors globally or from one point - Stack Overflow. Retrieved July 12, 2025, from [https://stackoverflow.com/questions/48990632/how-to-manage-axios-errors-globally-or-from-one-point](https://stackoverflow.com/questions/48990632/how-to-manage-axios-errors-globally-or-from-one-point)
15. Best Practices for Using Axios to Make HTTP Requests to a Spring Backend in a React Application | by Balian's Technologies and Innovation Lab. Retrieved July 12, 2025, from [https://medium.com/@ShantKhayalian/best-practices-for-using-axios-to-make-http-requests-to-a-spring-backend-in-a-react-application-f20d52ef41d3](https://medium.com/@ShantKhayalian/best-practices-for-using-axios-to-make-http-requests-to-a-spring-backend-in-a-react-application-f20d52ef41d3)
16. Token Refresh with Axios Interceptors for a Seamless Authentication Experience - Medium. Retrieved July 12, 2025, from [https://medium.com/@velja/token-refresh-with-axios-interceptors-for-a-seamless-authentication-experience-854b06064bde](https://medium.com/@velja/token-refresh-with-axios-interceptors-for-a-seamless-authentication-experience-854b06064bde)
17. A guide to handling refresh tokens with Axios - JavaScript in Plain English. Retrieved July 12, 2025, from [https://javascript.plainenglish.io/handle-refresh-token-with-axios-1e0f45e9afa](https://javascript.plainenglish.io/handle-refresh-token-with-axios-1e0f45e9afa)
18. Where to store token in local or session? : r/reactjs - Reddit. Retrieved July 12, 2025, from [https://www.reddit.com/r/reactjs/comments/1g8kc0z/where_to_store_token_in_local_or_session/](https://www.reddit.com/r/reactjs/comments/1g8kc0z/where_to_store_token_in_local_or_session/)
19. React SDK / SignOn Widget - Cookies vs. localStorage - Okta Developer Community. Retrieved July 12, 2025, from [https://devforum.okta.com/t/react-sdk-signon-widget-cookies-vs-localstorage/1144](https://devforum.okta.com/t/react-sdk-signon-widget-cookies-vs-localstorage/1144)
20. Are you making these JWT Authentication mistakes? - Duck Type Labs. Retrieved July 12, 2025, from [https://www.ducktypelabs.com/5-mistakes-web-developers-should-avoid-when-using-jwts-for-authentication/](https://www.ducktypelabs.com/5-mistakes-web-developers-should-avoid-when-using-jwts-for-authentication/)
21. Understanding Cookies and Sessions in React - SitePoint. Retrieved July 12, 2025, from [https://www.sitepoint.com/react-cookies-sessions/](https://www.sitepoint.com/react-cookies-sessions/)
22. React Cookies: A Guide to Managing Cookies in React Apps - CodeParrot. Retrieved July 12, 2025, from [https://codeparrot.ai/blogs/react-cookies-a-complete-guide-to-managing-cookies-in-your-react-application](https://codeparrot.ai/blogs/react-cookies-a-complete-guide-to-managing-cookies-in-your-react-application)
23. React Routing Guide - Strapi. Retrieved July 12, 2025, from [https://strapi.io/blog/react-routing-guide](https://strapi.io/blog/react-routing-guide)
24. Handling protected routes with React Router - Louis Young. Retrieved July 12, 2025, from [https://blog.louisyoung.co.uk/handling-protected-routes-react-router/](https://blog.louisyoung.co.uk/handling-protected-routes-react-router/)
25. Best Practices for Error Handling in React Context API State Management - MoldStud. Retrieved July 12, 2025, from [https://moldstud.com/articles/p-best-practices-for-error-handling-in-react-context-api-state-management](https://moldstud.com/articles/p-best-practices-for-error-handling-in-react-context-api-state-management)
26. 7 Ways to Avoid API Security Pitfalls when using JWT or JSON - 42Crunch. Retrieved July 12, 2025, from [https://42crunch.com/7-ways-to-avoid-jwt-pitfalls/](https://42crunch.com/7-ways-to-avoid-jwt-pitfalls/)