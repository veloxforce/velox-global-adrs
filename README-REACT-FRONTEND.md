<project_guide>
  <title>Simplified Project Structure Guide</title>
  
  <directory_structure>
    <section_title>Directory Structure</section_title>
    <code_block>
src/
├── features/                # Feature-based modules
│   └── [feature]/          # e.g., admin, calculator
│       ├── components/     # Feature-specific components (always flat, no subfolders)
│       │   └── index.js    # Barrel export file
│       ├── context/        # Feature-specific context providers
│       │   └── index.js    # Context barrel export
│       ├── hooks/          # Additional feature-specific hooks (if needed)
│       │   └── index.js    # Hooks barrel export
│       └── index.js        # Feature barrel export
│
├── shared/                 # Cross-feature shared code
│   ├── components/        # Shared UI components (always flat, no subfolders)
│   │   └── index.js       # Barrel export file
│   ├── data/              # ALL data access hooks go here
│   │   └── index.js       # Data hooks barrel export
│   └── index.js           # Shared barrel export
│
├── components/            # UI components
│   └── ui/                # shadcn/ui components (never modify these)
│       └── index.js       # Barrel export file
│
├── lib/                   # Library code
│   └── utils.js           # Utility functions
│
└── core/                  # Application essentials
    ├── Layout.jsx        # Layout component with context providers
    ├── router.js         # Single router configuration
    ├── services/         # External service clients folder
    │   └── index.js      # Services barrel export
    ├── state.js          # Application-wide state (when needed)
    └── index.js          # Core barrel export
    </code_block>
  </directory_structure>
  
  <import_rules>
    <section_title>Direct Import Rules</section_title>
    <description>For optimal code organization:</description>
    
    <rule>
      <rule_number>1</rule_number>
      <rule_title>**Direct Data Hook Imports**</rule_title>
      <rule_content>
        - Components and hooks should import shared data hooks directly:
        <code_block>
// CORRECT: Import directly from shared/data
import { useUserData } from '@/shared/data/useUserData';

// AVOID: Don't re-export shared data hooks through feature-level index files
// import { useUserData } from '@/features/admin/hooks'; ❌
        </code_block>
      </rule_content>
    </rule>
    
    <rule>
      <rule_number>2</rule_number>
      <rule_title>**Simple Dependency Chain**</rule_title>
      <rule_content>
        - Maintain a clean, direct dependency path
        - Feature components → Shared data hooks → Core services
      </rule_content>
    </rule>
  </import_rules>
  
  <decision_framework>
    <section_title>Decision Framework: Where Does My Code Go?</section_title>
    <description>When adding new code, follow this simplified decision process:</description>
    
    <infrastructure_decision>
      <subsection_title>1. Is this code infrastructure or configuration?</subsection_title>
      <content>
        If the code configures app-wide behavior or connects to external services, place it in `/core`.

        **Examples:** 
        - Router configuration → `/core/router.js`
        - API/database clients → `/core/services/[service-name].js`
        - Authentication setup → `/core/services/auth.js`
        - Global error handlers → `/core/error.js`
      </content>
    </infrastructure_decision>
    
    <ui_element_decision>
      <subsection_title>2. Is this UI element available in shadcn/ui?</subsection_title>
      <content>
        Check if it already exists in `/components/ui` - never modify these pre-configured components.

        **Examples:** Button, Input, Card, Dialog, Table
      </content>
    </ui_element_decision>
    
    <data_hook_decision>
      <subsection_title>3. Is this a data hook?</subsection_title>
      <content>
        ALL data hooks go in `/shared/data` regardless of how many features currently use them.

        **Examples:** User data, product data, configuration data
      </content>
    </data_hook_decision>
    
    <context_decision>
      <subsection_title>4. Is this feature state shared across components?</subsection_title>
      <content>
        If you need to share state between multiple components in a feature or we need to make the state resistant to route changes, create a context provider:

        **⚠️ CRITICAL: Context Placement Determines State Persistence**

        **For Component-Level State (Dies on Navigation):**
        Place it in `/features/[feature]/context/[Feature]Context.jsx`

        **For Cross-Route State (Survives Navigation):**
        Place it in `/core/Layout.jsx` using the Layout Component Pattern

        **Examples:**
        - User selection state shared between list and detail views → **Layout Level**
        - Form wizard state shared across multiple steps → **Layout Level**
        - Delete confirmation state shared between list and dialog → **Component Level**
        - State that needs to persist across route changes → **Layout Level**
      </content>
    </context_decision>
    
    <state_isolation_decision>
      <subsection_title>4a. When to isolate state from React Query?</subsection_title>
      <content>
        **Use Pure React Context State (useState):**
        - For UI state that needs to persist across route changes
        - For form state that should not be affected by API cache invalidations
        - When you need predictable state updates independent of network operations
        - For complex multi-step forms or wizards
        - When state needs to be completely isolated from server state
        
        **Use React Query Cache:**
        - For server-derived state that should be synchronized with the backend
        - When you want automatic cache invalidation and refetching
        - For data that should be consistent across the application
        - When optimistic updates are needed for better UX
        - For data that doesn't need to persist across route changes
        
        **Implementation Example:**
        ```jsx
        // Pure React Context State Example
        export function FeatureProvider({ children }) {
          // UI state lives directly in the context using useState
          const [formState, setFormState] = useState({
            field1: "",
            field2: "",
          });
          
          // Use React Query only for server state
          const { serverData, mutateData } = useDataHook();
          
          // Context value with both state types clearly separated
          const contextValue = {
            // UI State (managed by React)
            formState,
            setFormState,
            
            // Server State (managed by React Query)
            serverData,
            mutateData
          };
          
          return (
            <FeatureContext.Provider value={contextValue}>
              {children}
            </FeatureContext.Provider>
          );
        }
        ```
      </content>
    </state_isolation_decision>
    
    <component_decision>
      <subsection_title>5. Is this a new UI component?</subsection_title>
      <content>
        Place it in the feature you're currently working on: `/features/[feature]/components/`.
        Only move to `/shared/components` when actually used by a second feature.

        **Examples:** Feature-specific forms, cards, list items
      </content>
    </component_decision>
  </decision_framework>
  
  <context_placement_strategy>
    <section_title>Context Placement Strategy</section_title>

    <critical_rule>
      <subsection_title>CRITICAL RULE: Context Level = State Persistence</subsection_title>
      <content>
        **⚠️ Context placement determines whether state survives navigation:**

        | Placement Level | Survives Navigation | Router Context Access | Use Case |
        |----------------|-------------------|---------------------|----------|
        | **Component Level** | ❌ NO | ❌ NO | Form state, modal state |
        | **Layout Level** | ✅ YES | ✅ YES | Feature state, user selections |
        | **App Level** | ✅ YES | ❌ NO | Global auth, theme |

        **🎯 LAYOUT LEVEL IS PREFERRED** for most contexts because:
        - ✅ State persists across route navigation
        - ✅ Has access to `useNavigate()`, `useLocation()`, router hooks
        - ✅ Can trigger navigation from context logic
        - ✅ Perfect for cross-route feature state
      </content>
    </critical_rule>

    <navigation_warning>
      <subsection_title>⚠️ Navigation State Loss Warning</subsection_title>
      <content>
        **Component-level contexts DIE when navigating between routes.**

        If your context state needs to survive route changes (user selections, call state, form wizards), you **MUST** use the Layout Component Pattern.

        **❌ BROKEN: Component-level context**
        ```jsx
        // State DIES when navigating away from this route
        export function FeaturePage() {
          return (
            <FeatureProvider>  // ← DIES ON NAVIGATION
              <ComponentA />
              <ComponentB />
            </FeatureProvider>
          );
        }
        ```

        **✅ CORRECT: Layout-level context**
        ```jsx
        // State SURVIVES all navigation
        export function Layout() {
          return (
            <FeatureProvider>  // ← SURVIVES NAVIGATION + HAS ROUTER ACCESS
              <Outlet />       // ← Renders child routes
            </FeatureProvider>
          );
        }
        ```
      </content>
    </navigation_warning>
  </context_placement_strategy>

  <state_management>
    <section_title>State Management Approach</section_title>

    <component_state>
      <subsection_title>1. Component State</subsection_title>
      <content>
        **WHEN TO USE:** UI-only state for a single component

        <code_block>
function FeatureComponent() {
  const [isOpen, setIsOpen] = useState(false);
}
        </code_block>
      </content>
    </component_state>
    
    <feature_state>
      <subsection_title>2. Feature State: The Layout Component Pattern</subsection_title>
      <content>
        **WHEN TO USE:** State shared between components in a feature, business logic

        **🎯 RECOMMENDED: Use Layout Component Pattern for cross-route state persistence**

        <code_block>
// Step 1: Create context in features/admin/context/UserContext.jsx
import { createContext, useContext, useState } from 'react';
import { useNavigate } from 'react-router-dom';
import { useUserData } from '@/shared/data/useUserData';

const UserContext = createContext(null);

export function UserProvider({ children }) {
  const navigate = useNavigate(); // ✅ Available in Layout level

  // Local feature state
  const [selectedUserId, setSelectedUserId] = useState(null);

  // Use the shared data hook
  const { users, create, update, delete: deleteUser } = useUserData();

  // Add feature-specific business logic with navigation
  const createAdmin = (userData) => {
    create({ ...userData, role: 'admin' });
    navigate('/admin/success'); // ✅ Can navigate from context
  };

  // Transform or filter data for feature needs
  const activeAdmins = users?.filter(user =>
    user.role === 'admin' && user.status === 'active'
  ) || [];

  const contextValue = {
    users,
    selectedUserId,
    setSelectedUserId,
    activeAdmins,
    createAdmin,
    update,
    deleteUser
  };

  return (
    <UserContext.Provider value={contextValue}>
      {children}
    </UserContext.Provider>
  );
}

export function useUserContext() {
  const context = useContext(UserContext);
  if (!context) {
    throw new Error('useUserContext must be used within a UserProvider');
  }
  return context;
}

// Step 2: Use in Layout component (core/Layout.jsx)
import { Outlet } from 'react-router-dom';
import { UserProvider } from '@/features/admin/context/UserContext';

export function Layout() {
  return (
    <UserProvider>  // ✅ SURVIVES NAVIGATION + HAS ROUTER ACCESS
      <Outlet />    // ✅ Renders all child routes
    </UserProvider>
  );
}

// Step 3: Router configuration
export const router = createBrowserRouter([
  {
    path: "/",
    element: <Layout />,
    children: [
      { path: "admin", element: <AdminPage /> },
      { path: "admin/users", element: <UserList /> },
      { path: "admin/success", element: <SuccessPage /> },
    ],
  },
]);
        </code_block>
      </content>
    </feature_state>
    
    <application_state>
      <subsection_title>3. Application State</subsection_title>
      <content>
        **WHEN TO USE:** State accessed across multiple features

        For truly cross-feature state, use global context in `/core/state.js`
      </content>
    </application_state>
  </state_management>
  
  <data_hooks>
    <section_title>Data Hook Standardization</section_title>
    
    <shared_data_hooks>
      <subsection_title>1. Shared Data Hooks (in `/shared/data`)</subsection_title>
      <content>
        Use the naming convention `use[Entity]Data` for shared data hooks:

        <code_block>
// shared/data/useUserData.js
import { database } from '@/core/services';
import { toast } from '@/components/ui/sonner';

export function useUserData() {
  const queryClient = useQueryClient();

  // Query for data
  const { data: users, isLoading, error } = useQuery({
    queryKey: ["users"],
    queryFn: async () => {
      const { data, error } = await database.from("users").select("*");
      if (error) throw error;
      return data;
    }
  });

  // Create mutation
  const create = useMutation({
    mutationFn: async (newUser) => {
      const { data, error } = await database.from("users").insert(newUser).select().single();
      if (error) throw error;
      return data;
    },
    onSuccess: () => {
      queryClient.invalidateQueries(["users"]);
      toast.success("User created successfully");
    },
    onError: (error) => {
      toast.error(`Error: ${error.message}`);
    }
  });
  
  // Return both data and mutations
  return {
    // Data
    users,
    isLoading,
    error,
    
    // Mutations
    create: create.mutate,
    update: update.mutate,
    delete: delete.mutate,
    
    // States
    isCreating: create.isPending,
    isUpdating: update.isPending,
    isDeleting: delete.isPending
  };
}
        </code_block>
      </content>
    </shared_data_hooks>
    
    <component_usage>
      <subsection_title>2. Direct Usage in Components</subsection_title>
      <content>
        Components should import shared data hooks directly:

        <code_block>
// features/admin/components/UserList.jsx
import { useUserData } from '@/shared/data/useUserData';
import { Table } from '@/components/ui/table';

export function UserList() {
  const { users, isLoading } = useUserData();
  
  if (isLoading) return <div>Loading...</div>;
  
  return (
    <Table>
      {/* Table implementation */}
    </Table>
  );
}
        </code_block>
      </content>
    </component_usage>
    
    <context_usage>
      <subsection_title>3. Context Usage in Components</subsection_title>
      <content>
        Components should use the feature context through the custom hook:

        <code_block>
// features/admin/components/UserList.jsx
import { useUserContext } from '@/features/admin/context/UserContext';
import { Table } from '@/components/ui/table';

export function UserList() {
  const { users, isLoading, selectedUserId, setSelectedUserId } = useUserContext();
  
  if (isLoading) return <div>Loading...</div>;
  
  return (
    <Table>
      {users.map(user => (
        <tr 
          key={user.id} 
          className={user.id === selectedUserId ? 'selected' : ''}
          onClick={() => setSelectedUserId(user.id)}
        >
          <td>{user.name}</td>
        </tr>
      ))}
    </Table>
  );
}

// features/admin/components/AdminPage.jsx
import { useUserContext } from '@/features/admin/context/UserContext';
import { UserList } from './UserList';
import { UserDetails } from './UserDetails';

export function AdminPage() {
  // ✅ Context available from Layout level
  const { users, selectedUserId } = useUserContext();

  return (
    <div className="grid grid-cols-2 gap-4">
      <UserList />
      <UserDetails />
    </div>
  );
}
        </code_block>
      </content>
    </context_usage>

    <data_fetching_anti_patterns>
      <subsection_title>4. Data Fetching Anti-Patterns</subsection_title>
      <content>
        **❌ Direct API Calls in Components:**

        Never call API services directly from components. This bypasses React Query's caching, causes race conditions, and scatters data fetching logic.

        <code_block>
// BAD: Direct API call in component
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    apiClient.get(`/users/${userId}`).then(setUser);  // ❌
  }, [userId]);

  return <div>{user?.name}</div>;
}
        </code_block>

        **Why this is problematic:**
        - **Race conditions** - Multiple requests can resolve out of order
        - **No caching** - Same data fetched repeatedly across components
        - **Memory leaks** - State updates on unmounted components
        - **No loading/error states** - Must manually track fetch status
        - **Poor testability** - Fetch logic mixed with render logic
        - **Duplicated code** - Same fetch logic copied across components

        **✅ Use Data Hooks Instead:**

        <code_block>
// GOOD: Data hook encapsulates fetching
// shared/data/useUserData.js
export function useUserData(userId) {
  return useQuery({
    queryKey: ['user', userId],
    queryFn: () => apiClient.get(`/users/${userId}`),
  });
}

// Component just uses the hook
function UserProfile({ userId }) {
  const { data: user, isLoading } = useUserData(userId);

  if (isLoading) return <Spinner />;
  return <div>{user?.name}</div>;
}
        </code_block>

        **The Correct Dependency Chain:**
        ```
        Component → Shared Data Hook → Core Service → API
        ```

        **Sources:**
        - [You Might Not Need an Effect – React Docs](https://react.dev/learn/you-might-not-need-an-effect)
        - [TanStack Query Overview](https://tanstack.com/query/latest/docs/framework/react/overview)
      </content>
    </data_fetching_anti_patterns>

    <mutation_anti_patterns>
      <subsection_title>5. Data Hook Mutation Anti-Patterns</subsection_title>
      <content>
        **❌ Hardcoding Field Values in Mutations:**

        Never hardcode field values inside data hook mutations. Hooks are data access layers, not business logic layers. The caller should control what gets inserted.

        <code_block>
// BAD: Hook hardcodes business logic
export function useEntityData() {
  const create = useMutation({
    mutationFn: async (input) => {
      await database.from("entities").insert({
        ...input,
        is_predefined: false,  // ❌ Hardcoded - blocks admin use case
        user_id: user.id,      // ❌ Hardcoded - blocks system entities
      });
    },
  });
}
        </code_block>

        **Why this is problematic:**
        - **Blocks reuse** - Same hook can't serve admin (is_predefined=true) and user (is_predefined=false)
        - **Violates SRP** - Hook encodes business rules that belong in the caller
        - **Hidden behavior** - Caller doesn't know fields are being overwritten
        - **Testing difficulty** - Can't test different scenarios without modifying hook

        **✅ Caller Controls All Field Values:**

        <code_block>
// GOOD: Hook passes through what caller provides
export function useEntityData() {
  const create = useMutation({
    mutationFn: async (input) => {
      await database.from("entities").insert(input);  // ✅ Pass-through
    },
  });

  return { create: create.mutate };
}

// Caller controls values based on context
function AdminPage() {
  const { create } = useEntityData();
  const { isAdminMode } = useAdminMode();
  const { user } = useAuth();

  const handleCreate = (formData) => {
    create({
      ...formData,
      is_predefined: isAdminMode,              // ✅ Caller decides
      user_id: isAdminMode ? null : user.id,   // ✅ Caller decides
    });
  };
}
        </code_block>

        **The Principle:**
        ```
        Hooks = Data Access (WHAT to persist)
        Callers = Business Logic (HOW to use the hook)
        ```

        **Default Values (if needed):**
        If defaults are required, make them explicit and overridable:

        <code_block>
// ACCEPTABLE: Explicit defaults that caller can override
const create = useMutation({
  mutationFn: async (input) => {
    await database.from("entities").insert({
      is_predefined: false,  // Default, but...
      ...input,              // ...caller's values win
    });
  },
});
        </code_block>
      </content>
    </mutation_anti_patterns>
  </data_hooks>

  <error_monitoring>
    <section_title>Error Monitoring</section_title>
    <description>Global error capture ensures all API and render errors are reported to monitoring tools (Sentry, GlitchTip, etc.)</description>

    <required_setup>
      <subsection_title>Required Setup</subsection_title>
      <content>
        **1. Install SDK:**
        <code_block>
npm install @sentry/react
        </code_block>

        **2. Initialize BEFORE React renders (in main.tsx):**
        <code_block>
import * as Sentry from '@sentry/react';

// Initialize FIRST - before createRoot()
Sentry.init({
  dsn: 'YOUR_DSN',
  environment: import.meta.env.MODE,
});
        </code_block>

        **3. Configure QueryClient with global error handlers:**
        <code_block>
import { QueryClient, QueryCache, MutationCache } from '@tanstack/react-query';
import * as Sentry from '@sentry/react';

const queryClient = new QueryClient({
  queryCache: new QueryCache({
    onError: (error) => Sentry.captureException(error),
  }),
  mutationCache: new MutationCache({
    onError: (error) => Sentry.captureException(error),
  }),
});
        </code_block>

        **4. Wrap App in ErrorBoundary:**
        <code_block>
import * as Sentry from '@sentry/react';

function App() {
  return (
    &lt;Sentry.ErrorBoundary fallback={&lt;ErrorFallback /&gt;}&gt;
      &lt;RouterProvider router={router} /&gt;
    &lt;/Sentry.ErrorBoundary&gt;
  );
}
        </code_block>
      </content>
    </required_setup>

    <what_gets_captured>
      <subsection_title>What Gets Captured</subsection_title>
      <content>
        | Error Type | Source | Captured By |
        |------------|--------|-------------|
        | API query errors | Network failures, 4xx, 5xx | QueryCache onError |
        | API mutation errors | POST/PUT/DELETE failures | MutationCache onError |
        | React render errors | Component crashes | Sentry.ErrorBoundary |
      </content>
    </what_gets_captured>

    <anti_patterns>
      <subsection_title>Anti-patterns to Avoid</subsection_title>
      <content>
        **❌ Toast-only error handling:**
        <code_block>
// BAD: Error is "handled" - monitoring never sees it
onError: (error) => toast.error(error.message)
        </code_block>

        **✅ Global handlers ensure capture:**
        The QueryCache/MutationCache onError handlers fire for ALL queries/mutations, even when individual hooks have their own onError. Your existing toast pattern continues to work - global handlers run first.

        **❌ No ErrorBoundary:**
        <code_block>
// BAD: React render errors crash app with white screen
&lt;App /&gt;
        </code_block>

        **✅ ErrorBoundary catches render errors:**
        <code_block>
// GOOD: Render errors caught, reported, fallback UI shown
&lt;Sentry.ErrorBoundary fallback={&lt;ErrorUI /&gt;}&gt;
  &lt;App /&gt;
&lt;/Sentry.ErrorBoundary&gt;
        </code_block>
      </content>
    </anti_patterns>
  </error_monitoring>

  <routing>
    <section_title>Routing: Layout Component Pattern (Recommended)</section_title>
    <content>
      **🎯 ALWAYS use Layout component pattern for context providers:**

      <code_block>
// core/Layout.jsx
import { Outlet } from 'react-router-dom';
import { FeatureProvider } from '@/features/admin/context';

export function Layout() {
  return (
    <FeatureProvider>  // ✅ Context persists across navigation
      <Outlet />       // ✅ Renders child routes
    </FeatureProvider>
  );
}

// core/router.js
import { Layout } from './Layout';
import { AdminDashboard, UserList } from '@/features/admin/components';
import { Calculator } from '@/features/calculator/components';

export const router = createBrowserRouter([
  {
    path: "/",
    element: <Layout />,  // ✅ Single context instance
    children: [
      { path: "admin", element: <AdminDashboard /> },
      { path: "admin/users", element: <UserList /> },
      { path: "calculator", element: <Calculator /> },
    ],
  },
]);
      </code_block>

      **Benefits:**
      - ✅ Context state persists across navigation
      - ✅ Router hooks available in context providers
      - ✅ Single provider instance for entire app
      - ✅ Clean separation of concerns

      Split into feature-based routing only when router file exceeds ~100 lines.
    </content>
  </routing>
  
  <naming_conventions>
    <section_title>File Naming Conventions</section_title>
    <content>
      1. **Components:** Use PascalCase (`FactorForm.jsx`)
      2. **Context Providers:** Use PascalCase with 'Context' suffix (`UserContext.jsx`)
      3. **Context Hooks:** Use camelCase with 'use[Feature]Context' pattern (`useUserContext`)
      4. **Data Hooks:** Use camelCase with 'use[Entity]Data' pattern (`useUserData.js`)
      5. **Utility Hooks:** Use camelCase with 'use' prefix (`useMediaQuery.js`)
      6. **Utility Functions:** Use camelCase (`formatDate.js`)
      7. **Barrel Files:** Always name `index.js`
    </content>
  </naming_conventions>
  
  <ai_assistant>
    <section_title>AI Implementation Assistant</section_title>
    <content>
      When working with AI assistance:

      1. The AI will suggest a directory location for new code by saying:
         "I recommend placing this code in [location] because [reasoning]."

      2. The AI must wait for explicit confirmation before generating or placing code.

      3. If uncertain, the AI should present options:
         "This could belong in either:
         - `/features/admin` if it's specific to admin functionality
         - [...]?
         Which would you prefer?"
    </content>
  </ai_assistant>
</project_guide>
