# Claude Generated Code

**Task**: Objective: Help design and implement new features based on the current roadmap or project needs. Focus on clean, scalable, and testable code  Claude Tasks:  🧩 Feature Implementation Read the existing codebase and integrate new feature modules where appropriate.  Ensure new code follows existing patterns and conventions.  🗂️ Example Features (replace/add specifics as needed): Add user role-based access control  Implement export to PDF  Add multi-language support  Enable dark mode toggle  🧪 Testing & Documentation Add unit/integration tests for new functionality.  Update README or in-code comments if required.

**Generated on**: Sun Jul 27 15:10:18 UTC 2025

## Generated Content

```
Looking at the codebase, I'll implement user role-based access control for the government billing/invoicing application. This will include user authentication, role management, and access restrictions for different invoice templates and functionalities.

## FILES_TO_CREATE

1. `src/types/auth.ts` - Type definitions for authentication
2. `src/types/roles.ts` - Role definitions and permissions
3. `src/hooks/useAuth.ts` - Authentication hook
4. `src/hooks/useRoles.ts` - Role management hook
5. `src/components/Auth/LoginForm.tsx` - Login component
6. `src/components/Auth/ProtectedRoute.tsx` - Route protection component
7. `src/components/Auth/RoleGuard.tsx` - Role-based component guard
8. `src/services/authService.ts` - Authentication service
9. `src/services/roleService.ts` - Role management service
10. `src/contexts/AuthContext.tsx` - Authentication context
11. `src/utils/permissions.ts` - Permission utilities

## FILES_TO_MODIFY

1. `src/App/App.js` - Add authentication wrapper
2. `src/Menu/Menu.js` - Add role-based menu filtering
3. `src/Files/Files.js` - Add access control for file operations
4. `package.json` - Add new dependencies

## CODE_CHANGES

### FILES_TO_CREATE

#### `src/types/auth.ts`
```typescript
export interface User {
  id: string;
  email: string;
  name: string;
  role: UserRole;
  department?: string;
  permissions: Permission[];
  createdAt: string;
  lastLogin?: string;
}

export interface LoginCredentials {
  email: string;
  password: string;
}

export interface AuthResponse {
  user: User;
  token: string;
  refreshToken: string;
}

export interface AuthState {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  error: string | null;
}
```

#### `src/types/roles.ts`
```typescript
export enum UserRole {
  ADMIN = 'admin',
  FINANCE_MANAGER = 'finance_manager',
  ACCOUNTANT = 'accountant',
  CLERK = 'clerk',
  VIEWER = 'viewer'
}

export enum Permission {
  // Invoice permissions
  CREATE_INVOICE = 'create_invoice',
  EDIT_INVOICE = 'edit_invoice',
  DELETE_INVOICE = 'delete_invoice',
  VIEW_INVOICE = 'view_invoice',
  APPROVE_INVOICE = 'approve_invoice',
  
  // Template permissions
  CREATE_TEMPLATE = 'create_template',
  EDIT_TEMPLATE = 'edit_template',
  DELETE_TEMPLATE = 'delete_template',
  VIEW_TEMPLATE = 'view_template',
  
  // File permissions
  UPLOAD_FILE = 'upload_file',
  DELETE_FILE = 'delete_file',
  SHARE_FILE = 'share_file',
  
  // User management
  MANAGE_USERS = 'manage_users',
  VIEW_USERS = 'view_users',
  
  // Reports
  VIEW_REPORTS = 'view_reports',
  EXPORT_REPORTS = 'export_reports',
  
  // System
  SYSTEM_SETTINGS = 'system_settings',
  AUDIT_LOGS = 'audit_logs'
}

export interface RolePermissions {
  [UserRole.ADMIN]: Permission[];
  [UserRole.FINANCE_MANAGER]: Permission[];
  [UserRole.ACCOUNTANT]: Permission[];
  [UserRole.CLERK]: Permission[];
  [UserRole.VIEWER]: Permission[];
}

export const DEFAULT_ROLE_PERMISSIONS: RolePermissions = {
  [UserRole.ADMIN]: [
    Permission.CREATE_INVOICE,
    Permission.EDIT_INVOICE,
    Permission.DELETE_INVOICE,
    Permission.VIEW_INVOICE,
    Permission.APPROVE_INVOICE,
    Permission.CREATE_TEMPLATE,
    Permission.EDIT_TEMPLATE,
    Permission.DELETE_TEMPLATE,
    Permission.VIEW_TEMPLATE,
    Permission.UPLOAD_FILE,
    Permission.DELETE_FILE,
    Permission.SHARE_FILE,
    Permission.MANAGE_USERS,
    Permission.VIEW_USERS,
    Permission.VIEW_REPORTS,
    Permission.EXPORT_REPORTS,
    Permission.SYSTEM_SETTINGS,
    Permission.AUDIT_LOGS
  ],
  [UserRole.FINANCE_MANAGER]: [
    Permission.CREATE_INVOICE,
    Permission.EDIT_INVOICE,
    Permission.DELETE_INVOICE,
    Permission.VIEW_INVOICE,
    Permission.APPROVE_INVOICE,
    Permission.VIEW_TEMPLATE,
    Permission.UPLOAD_FILE,
    Permission.SHARE_FILE,
    Permission.VIEW_USERS,
    Permission.VIEW_REPORTS,
    Permission.EXPORT_REPORTS
  ],
  [UserRole.ACCOUNTANT]: [
    Permission.CREATE_INVOICE,
    Permission.EDIT_INVOICE,
    Permission.VIEW_INVOICE,
    Permission.VIEW_TEMPLATE,
    Permission.UPLOAD_FILE,
    Permission.VIEW_REPORTS
  ],
  [UserRole.CLERK]: [
    Permission.CREATE_INVOICE,
    Permission.EDIT_INVOICE,
    Permission.VIEW_INVOICE,
    Permission.VIEW_TEMPLATE,
    Permission.UPLOAD_FILE
  ],
  [UserRole.VIEWER]: [
    Permission.VIEW_INVOICE,
    Permission.VIEW_TEMPLATE
  ]
};
```

#### `src/contexts/AuthContext.tsx`
```typescript
import React, { createContext, useContext, useReducer, useEffect } from 'react';
import { AuthState, User, LoginCredentials } from '../types/auth';
import { authService } from '../services/authService';

interface AuthContextType extends AuthState {
  login: (credentials: LoginCredentials) => Promise<void>;
  logout: () => void;
  refreshToken: () => Promise<void>;
  clearError: () => void;
}

const AuthContext = createContext<AuthContextType | null>(null);

type AuthAction =
  | { type: 'LOGIN_START' }
  | { type: 'LOGIN_SUCCESS'; payload: { user: User; token: string } }
  | { type: 'LOGIN_FAILURE'; payload: string }
  | { type: 'LOGOUT' }
  | { type: 'REFRESH_TOKEN_SUCCESS'; payload: { user: User; token: string } }
  | { type: 'CLEAR_ERROR' }
  | { type: 'SET_LOADING'; payload: boolean };

const authReducer = (state: AuthState, action: AuthAction): AuthState => {
  switch (action.type) {
    case 'LOGIN_START':
      return { ...state, isLoading: true, error: null };
    case 'LOGIN_SUCCESS':
    case 'REFRESH_TOKEN_SUCCESS':
      return {
        ...state,
        user: action.payload.user,
        token: action.payload.token,
        isAuthenticated: true,
        isLoading: false,
        error: null
      };
    case 'LOGIN_FAILURE':
      return {
        ...state,
        user: null,
        token: null,
        isAuthenticated: false,
        isLoading: false,
        error: action.payload
      };
    case 'LOGOUT':
      return {
        ...state,
        user: null,
        token: null,
        isAuthenticated: false,
        isLoading: false,
        error: null
      };
    case 'CLEAR_ERROR':
      return { ...state, error: null };
    case 'SET_LOADING':
      return { ...state, isLoading: action.payload };
    default:
      return state;
  }
};

const initialState: AuthState = {
  user: null,
  token: null,
  isAuthenticated: false,
  isLoading: true,
  error: null
};

export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [state, dispatch] = useReducer(authReducer, initialState);

  useEffect(() => {
    const initAuth = async () => {
      try {
        const token = localStorage.getItem('authToken');
        if (token) {
          const user = await authService.verifyToken(token);
          dispatch({ type: 'LOGIN_SUCCESS', payload: { user, token } });
        }
      } catch (error) {
        localStorage.removeItem('authToken');
        localStorage.removeItem('refreshToken');
      } finally {
        dispatch({ type: 'SET_LOADING', payload: false });
      }
    };

    initAuth();
  }, []);

  const login = async (credentials: LoginCredentials) => {
    dispatch({ type: 'LOGIN_START' });
    try {
      const response = await authService.login(credentials);
      localStorage.setItem('authToken', response.token);
      localStorage.setItem('refreshToken', response.refreshToken);
      dispatch({ type: 'LOGIN_SUCCESS', payload: response });
    } catch (error) {
      dispatch({ type: 'LOGIN_FAILURE', payload: error.message });
      throw error;
    }
  };

  const logout = async () => {
    try {
      await authService.logout();
    } catch (error) {
      console.error('Logout error:', error);
    } finally {
      localStorage.removeItem('authToken');
      localStorage.removeItem('refreshToken');
      dispatch({ type: 'LOGOUT' });
    }
  };

  const refreshToken = async () => {
    try {
      const refreshToken = localStorage.getItem('refreshToken');
      if (!refreshToken) throw new Error('No refresh token');

      const response = await authService.refreshToken(refreshToken);
      localStorage.setItem('authToken', response.token);
      dispatch({ type: 'REFRESH_TOKEN_SUCCESS', payload: response });
    } catch (error) {
      logout();
      throw error;
    }
  };

  const clearError = () => {
    dispatch({ type: 'CLEAR_ERROR' });
  };

  return (
    <AuthContext.Provider
      value={{
        ...state,
        login,
        logout,
        refreshToken,
        clearError
      }}
    >
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};
```

#### `src/hooks/useAuth.ts`
```typescript
import { useContext } from 'react';
import { AuthContext } from '../contexts/AuthContext';

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};
```

#### `src/hooks/useRoles.ts`
```typescript
import { useMemo } from 'react';
import { useAuth } from './useAuth';
import { Permission, UserRole } from '../types/roles';
import { hasPermission, canAccessTemplate } from '../utils/permissions';

export const useRoles = () => {
  const { user } = useAuth();

  const permissions = useMemo(() => {
    return user?.permissions || [];
  }, [user]);

  const checkPermission = (permission: Permission): boolean => {
    if (!user) return false;
    return hasPermission(user, permission);
  };

  const checkMultiplePermissions = (requiredPermissions: Permission[], requireAll = false): boolean => {
    if (!user) return false;
    
    if (requireAll) {
      return requiredPermissions.every(permission => hasPermission(user, permission));
    }
    
    return requiredPermissions.some(permission => hasPermission(user, permission));
  };

  const canAccessInvoiceTemplate = (templateType: string): boolean => {
    if (!user) return false;
    return canAccessTemplate(user, templateType);
  };

  const isAdmin = useMemo(() => {
    return user?.role === UserRole.ADMIN;
  }, [user]);

  const isFinanceManager = useMemo(() => {
    return user?.role === UserRole.FINANCE_MANAGER;
  }, [user]);

  const isAccountant = useMemo(() => {
    return user?.role === UserRole.ACCOUNTANT;
  }, [user]);

  const isClerk = useMemo(() => {
    return user?.role === UserRole.CLERK;
  }, [user]);

  const isViewer = useMemo(() => {
    return user?.role === UserRole.VIEWER;
  }, [user]);

  const canCreateInvoice = useMemo(() => {
    return checkPermission(Permission.CREATE_INVOICE);
  }, [permissions]);

  const canEditInvoice = useMemo(() => {
    return checkPermission(Permission.EDIT_INVOICE);
  }, [permissions]);

  const canDeleteInvoice = useMemo(() => {
    return checkPermission(Permission.DELETE_INVOICE);
  }, [permissions]);

  const canApproveInvoice = useMemo(() => {
    return checkPermission(Permission.APPROVE_INVOICE);
  }, [permissions]);

  const canManageUsers = useMemo(() => {
    return checkPermission(Permission.MANAGE_USERS);
  }, [permissions]);

  const canViewReports = useMemo(() => {
    return checkPermission(Permission.VIEW_REPORTS);
  }, [permissions]);

  const canExportReports = useMemo(() => {
    return checkPermission(Permission.EXPORT_REPORTS);
  }, [permissions]);

  return {
    permissions,
    checkPermission,
    checkMultiplePermissions,
    canAccessInvoiceTemplate,
    isAdmin,
    isFinanceManager,
    isAccountant,
    isClerk,
    isViewer,
    canCreateInvoice,
    canEditInvoice,
    canDeleteInvoice,
    canApproveInvoice,
    canManageUsers,
    canViewReports,
    canExportReports
  };
};
```

#### `src/services/authService.ts`
```typescript
import { LoginCredentials, AuthResponse, User } from '../types/auth';
import { UserRole, Permission, DEFAULT_ROLE_PERMISSIONS } from '../types/roles';

// Mock service - replace with actual API calls
class AuthService {
  private mockUsers: User[] = [
    {
      id: '1',
      email: 'admin@gov.com',
      name: 'System Administrator',
      role: UserRole.ADMIN,
      department: 'IT',
      permissions: DEFAULT_ROLE_PERMISSIONS[UserRole.ADMIN],
      createdAt: new Date().toISOString()
    },
    {
      id: '2',
      email: 'finance@gov.com',
      name: 'Finance Manager',
      role: UserRole.FINANCE_MANAGER,
      department: 'Finance',
      permissions: DEFAULT_ROLE_PERMISSIONS[UserRole.FINANCE_MANAGER],
      createdAt: new Date().toISOString()
    },
    {
      id: '3',
      email: 'accountant@gov.com',
      name: 'Senior Accountant',
      role: UserRole.ACCOUNTANT,
      department: 'Finance',
      permissions: DEFAULT_ROLE_PERMISSIONS[UserRole.ACCOUNTANT],
      createdAt: new Date().toISOString()
    },
    {
      id: '4',
      email: 'clerk@gov.com',
      name: 'Billing Clerk',
      role: UserRole.CLERK,
      department: 'Billing',
      permissions: DEFAULT_ROLE_PERMISSIONS[UserRole.CLERK],
      createdAt: new Date().toISOString()
    },
    {
      id: '5',
      email: 'viewer@gov.com',
      name: 'Report Viewer',
      role: UserRole.VIEWER,
      department: 'Management',
      permissions: DEFAULT_ROLE_PERMISSIONS[UserRole.VIEWER],
      createdAt: new Date().toISOString()
    }
  ];

  async login(credentials: LoginCredentials): Promise<AuthResponse> {
    // Simulate API delay
    await new Promise(resolve => setTimeout(resolve, 1000));

    const user = this.mockUsers.find(u => u.email === credentials.email);
    
    if (!user) {
      throw new Error('Invalid email or password');
    }

    // In a real app, you would verify the password hash
    if (credentials.password !== 'password123') {
      throw new Error('Invalid email or password');
    }

    const token = this.generateToken();
    const refreshToken = this.generateToken();

    // Update last login
    user.lastLogin = new Date().toISOString();

    return {
      user,
      token,
      refreshToken
    };
  }

  async logout(): Promise<void> {
    // Simulate API call
    await new Promise(resolve => setTimeout(resolve, 500));
    // In a real app, you would invalidate the token on the server
  }

  async verifyToken(token: string): Promise<User> {
    // Simulate API delay
    await new Promise(resolve => setTimeout(resolve, 500));

    // In a real app, you would verify the token with your backend
    if (!token || token === 'invalid') {
      throw new Error('Invalid token');
    }

    // Return mock user for demo
    return this.mockUsers[0];
  }

  async refreshToken(refreshToken: string): Promise<AuthResponse> {
    // Simulate API delay
    await new Promise(resolve => setTimeout(resolve, 500));

    if (!refreshToken) {
      throw new Error('Invalid refresh token');
    }

    // In a real app, you would verify the refresh token
    const user = this.mockUsers[0];
    const newToken = this.generateToken();
    const newRefreshToken = this.generateToken();

    return {
      user,
      token: newToken,
      refreshToken: newRefreshToken
    };
  }

  private generateToken(): string {
    return btoa(Math.random().toString(36).substr(2, 9));
  }
}

export const authService = new AuthService();
```

#### `src/services/roleService.ts`
```typescript
import { User, UserRole, Permission } from '../types/roles';

class RoleService {
  async updateUserRole(userId: string, role: UserRole): Promise<User> {
    // Simulate API call
    await new Promise(resolve => setTimeout(resolve, 500));
    
    // In a real app, you would make an API call to update the user's role
    throw new Error('Not implemented');
  }

  async updateUserPermissions(userId: string, permissions: Permission[]): Promise<User> {
    // Simulate API call
    await new Promise(resolve => setTimeout(resolve, 500));
    
    // In a real app, you would make an API call to update the user's permissions
    throw new Error('Not implemented');
  }

  async getUsers(): Promise<User[]> {
    // Simulate API call
    await new Promise(resolve => setTimeout(resolve, 500));
    
    // In a real app, you would fetch users from your backend
    throw new Error('Not implemented');
  }
}

export const roleService = new RoleService();
```

#### `src/utils/permissions.ts`
```typescript
import { User, Permission, UserRole } from '../types/roles';

export const hasPermission = (user: User, permission: Permission): boolean => {
  if (!user || !user.permissions) return false;
  return user.permissions.includes(permission);
};

export const hasAnyPermission = (user: User, permissions: Permission[]): boolean => {
  if (!user || !user.permissions) return false;
  return permissions.some(permission => user.permissions.includes(permission));
};

export const hasAllPermissions = (user: User, permissions: Permission[]): boolean => {
  if (!user || !user.permissions) return false;
  return permissions.every(permission => user.permissions.includes(permission));
};

export const canAccessTemplate = (user: User, templateType: string): boolean => {
  if (!user) return false;

  // Admin can access all templates
  if (user.role === UserRole.ADMIN) return true;

  // Define template access rules
  const templateAccess = {
    'typei': [UserRole.ADMIN, UserRole.FINANCE_MANAGER, UserRole.ACCOUNTANT, UserRole.CLERK],
    'typeii': [UserRole.ADMIN, UserRole.FINANCE_MANAGER, UserRole.ACCOUNTANT],
    'typeiii': [UserRole.ADMIN, UserRole.FINANCE_MANAGER],
    'typeiv': [UserRole.ADMIN, UserRole.FINANCE_MANAGER]
  };

  const allowedRoles = templateAccess[templateType];
  return allowedRoles ? allowedRoles.includes(user.role) : false;
};

export const getAccessibleTemplates = (user: User): string[] => {
  if (!user) return [];

  const allTemplates = ['typei', 'typeii', 'typeiii', 'typeiv'];
  return allTemplates.filter(template => canAccessTemplate(user, template));
};

export const canEditInvoiceAmount = (user: User, amount: number): boolean => {
  if (!user) return false;

  // Different roles have different amount limits
  const amountLimits = {
    [UserRole.ADMIN]: Infinity,
    [UserRole.FINANCE_MANAGER]: 100000,
    [UserRole.ACCOUNTANT]: 50000,
    [UserRole.CLERK]: 10000,
    [UserRole.VIEWER]: 0
  };

  const limit = amountLimits[user.role] || 0;
  return amount <= limit;
};
```

#### `src/components/Auth/LoginForm.tsx`
```typescript
import React, { useState } from 'react';
import {
  IonButton,
  IonCard,
  IonCardContent,
  IonCardHeader,
  IonCardTitle,
  IonInput,
  IonItem,
  IonLabel,
  IonNote,
  IonSpinner,
  IonAlert,
  IonGrid,
  IonRow,
  IonCol,
  IonIcon
} from '@ionic/react';
import { logInOutline, personOutline } from 'ionicons/icons';
import { useAuth } from '../../hooks/useAuth';

const LoginForm: React.FC = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [showError, setShowError] = useState(false);
  const { login, isLoading, error } = useAuth();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    try {
      await login({ email, password });
    } catch (err) {
      setShowError(true);
    }
  };

  return (
    <IonGrid>
      <IonRow className="ion-justify-content-center">
        <IonCol size="12" sizeMd="6" sizeLg="4">
          <IonCard>
            <IonCardHeader>
              <IonCardTitle className="ion-text-center">
                <IonIcon icon={logInOutline} size="large" />
                <h2>Government Invoice System</h2>
              </IonCardTitle>
            </IonCardHeader>
            <IonCardContent>
              <form onSubmit={handleSubmit}>
                <IonItem>
                  <IonLabel position="stacked">Email</IonLabel>
                  <IonInput
                    type="email"
                    value={email}
                    onIonInput={(e) => setEmail(e.detail.value!)}
                    required
                    placeholder="Enter your email"
                  />
                </IonItem>

                <IonItem>
                  <IonLabel position="stacked">Password</IonLabel>
                  <IonInput
                    type="password"
                    value={password}
                    onIonInput={(e) => setPassword(e.detail.value!)}
                    required
                    placeholder="Enter your password"
                  />
                </IonItem>

                <IonButton
                  expand="full"
                  type="submit"
                  disabled={isLoading || !email || !password}
                  className="ion-margin-top"
                >
                  {isLoading ? <IonSpinner /> : 'Login'}
                </IonButton>
              </form>

              <div className="ion-margin-top">
                <IonNote>
                  <h4>Demo Accounts:</h4>
                  <p>Admin: admin@gov.com / password123</p>
                  <p>Finance Manager: finance@gov.com / password123</p>
                  <p>Accountant: accountant@gov.com / password123</p>
                  <p>Clerk: clerk@gov.com / password123</p>
                  <p>Viewer: viewer@gov.com / password123</p>
                </IonNote>
              </div>
            </IonCardContent>
          </IonCard>
        </IonCol>
      </IonRow>

      <IonAlert
        isOpen={showError}
        onDidDismiss={() => setShowError(false)}
        header="Login Error"
        message={error || 'An error occurred during login'}
        buttons={['OK']}
      />
    </IonGrid>
  );
};

export default LoginForm;
```

#### `src/components/Auth/ProtectedRoute.tsx`
```typescript
import React from 'react';
import { IonSpinner, IonContent } from '@ionic/react';
import { useAuth } from '../../hooks/useAuth';
import LoginForm from './LoginForm';

interface ProtectedRouteProps {
  children: React.ReactNode;
}

const ProtectedRoute: React.FC<ProtectedRouteProps> = ({ children }) => {
  const { isAuthenticated, isLoading } = useAuth();

  if (isLoading) {
    return (
      <IonContent className="ion-padding">
        <div className="ion-text-center ion-padding">
          <IonSpinner />
          <p>Loading...</p>
        </div>
      </IonContent>
    );
  }

  if (!isAuthenticated) {
    return (
      <IonContent className="ion-padding">
        <LoginForm />
      </IonContent>
    );
  }

  return <>{children}</>;
};

export default ProtectedRoute;
```

#### `src/components/Auth/RoleGuard.tsx`
```typescript
import React from 'react';
import { IonButton, IonCard, IonCardContent, IonCardHeader, IonCardTitle, IonIcon } from '@ionic/react';
import { lockClosedOutline } from 'ionicons/icons';
import { Permission } from '../../types/roles';
import { useRoles } from '../../hooks/useRoles';

interface RoleGuardProps {
  children: React.ReactNode;
  permission?: Permission;
  permissions?: Permission[];
  requireAll?: boolean;
  fallback?: React.ReactNode;
  showFallback?: boolean;
}

const RoleGuard: React.FC<RoleGuardProps> = ({
  children,
  permission,
  permissions,
  requireAll = false,
  fallback,
  showFallback = true
}) => {
  const { checkPermission, checkMultiplePermissions } = useRoles();

  let hasAccess = false;

  if (permission) {
    hasAccess = checkPermission(permission);
  } else if (permissions) {
    hasAccess = checkMultiplePermissions(permissions, requireAll);
  }

  if (!hasAccess) {
    if (fallback) {
      return <>{fallback}</>;
    }

    if (showFallback) {
      return (
        <IonCard>
          <IonCardHeader>
            <IonCardTitle className="ion-text-center">
              <IonIcon icon={lockClosedOutline} size="large" />
              <h3>Access Denied</h3>
            </IonCardTitle>
          </IonCardHeader>
          <IonCardContent className="ion-text-center">
            <p>You don't have permission to access this feature.</p>
            <p>Please contact your administrator if you need access.</p>
          </IonCardContent>
        </IonCard>
      );
    }

    return null;
  }

  return <>{children}</>;
};

export default RoleGuard;
```

### FILES_TO_MODIFY

#### `package.json`
```json
{
  "name": "react-webapp",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "@ionic/react": "^7.0.0",
    "@ionic/react-router": "^7.0.0",
    "@tanstack/react-query": "^5.49.2",
    "@testing-library/jest-dom": "^5.17.0",
    "@testing-library/react": "^13.4.0",
    "@testing-library/user-event": "^13.5.0",
    "@types/react": "^18.0.0",
    "@types/react-dom": "^18.0.0",
    "@web3-storage/w3up-client": "^17.2.0",
    "connectkit": "^1.8.2",
    "ethers": "5.6.0",
    "ionicons": "^7.0.0",
    "react": "^18.3.1",
    "react-dom": "^18.3.1",
    "react-router": "^6.0.0",
    "react-router-dom": "^6.0.0",
    "react-scripts": "5.0.1",
    "typescript": "^4.9.0",
    "viem": "2.x",
    "wagmi": "^2.10.9",
    "web-vitals": "^2.1.4",
    "whatwg-fetch": "^2.0.3"
  },
  "devDependencies": {
    "@babel/plugin-proposal-private-property-in-object": "^7.21.11",
    "@types/node": "^18.0.0",
    "react-scripts": "5.0.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  },
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  }
}
```

#### `src/App/App.js` (convert to TypeScript and add auth)
```typescript
import React from "react";
import { IonApp, IonRouterOutlet, setupIonicReact } from '@ionic/react';
import { IonReactRouter } from '@ionic/react-router';
import { Route } from 'react-router-dom';
import "./App.css";
import { AuthProvider }
```
