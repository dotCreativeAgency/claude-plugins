# React Stack Patterns

Advanced patterns for TanStack Query v5, React Hook Form + Zod, React Router v7, and state management.

## TanStack Query v5

### Query Patterns

#### Basic Query with Error Handling

```tsx
import { useQuery } from '@tanstack/react-query';
import { apiClient } from '@api/client'; // Project-specific API client - adjust import path to your project

interface UseUsersOptions {
  page?: number;
  pageSize?: number;
  search?: string;
}

export function useUsers({ page = 1, pageSize = 25, search }: UseUsersOptions = {}) {
  return useQuery({
    queryKey: ['users', { page, pageSize, search }],
    queryFn: async () => {
      const { data } = await apiClient.get('/api/v1/users', {
        params: { page, per_page: pageSize, search },
      });
      return data;
    },
    staleTime: 5 * 60 * 1000, // 5 minutes
    placeholderData: (previousData) => previousData, // Keep previous data while fetching
  });
}
```

#### Dependent Queries

```tsx
const { data: user } = useQuery({
  queryKey: ['user', userId],
  queryFn: () => fetchUser(userId),
  enabled: !!userId,
});

const { data: permissions } = useQuery({
  queryKey: ['permissions', user?.role],
  queryFn: () => fetchPermissions(user!.role),
  enabled: !!user?.role, // Only fetch when user is loaded
});
```

#### Prefetching

```tsx
const queryClient = useQueryClient();

// Prefetch on hover
const handleMouseEnter = () => {
  queryClient.prefetchQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
    staleTime: 60 * 1000,
  });
};
```

### Mutation Patterns

#### Optimistic Updates

```tsx
const mutation = useMutation({
  mutationFn: updateUser,
  onMutate: async (newData) => {
    await queryClient.cancelQueries({ queryKey: ['users'] });
    const previousUsers = queryClient.getQueryData(['users']);
    queryClient.setQueryData(['users'], (old) =>
      old?.map((u) => (u.id === newData.id ? { ...u, ...newData } : u))
    );
    return { previousUsers };
  },
  onError: (_err, _newData, context) => {
    queryClient.setQueryData(['users'], context?.previousUsers);
  },
  onSettled: () => {
    queryClient.invalidateQueries({ queryKey: ['users'] });
  },
});
```

#### Delete with Confirmation

```tsx
const deleteMutation = useMutation({
  mutationFn: (id: string) => apiClient.delete(`/api/v1/users/${id}`),
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['users'] });
    enqueueSnackbar('User deleted successfully', { variant: 'success' });
  },
  onError: (error) => {
    enqueueSnackbar('Failed to delete user', { variant: 'error' });
  },
});
```

### Query Key Conventions

```tsx
// Entity list
queryKey: ['users']
queryKey: ['users', { page, search, filters }]

// Single entity
queryKey: ['users', userId]

// Nested resource
queryKey: ['users', userId, 'permissions']

// Invalidation patterns
queryClient.invalidateQueries({ queryKey: ['users'] }); // All user queries
queryClient.invalidateQueries({ queryKey: ['users', userId] }); // Specific user
```

### Custom Query Hook Pattern

```tsx
export function useEntityQuery<T>(
  entityName: string,
  id: string | undefined,
  fetchFn: (id: string) => Promise<T>
) {
  return useQuery({
    queryKey: [entityName, id],
    queryFn: () => fetchFn(id!),
    enabled: !!id,
  });
}

// Usage
const { data: tenant } = useEntityQuery('tenants', tenantId, fetchTenant);
```

## React Hook Form + Zod

### Schema Patterns

#### Basic Schema

```tsx
import { z } from 'zod';

const userSchema = z.object({
  name: z.string().min(1, 'Name is required').max(100),
  email: z.string().email('Invalid email address'),
  role: z.enum(['admin', 'user', 'editor']),
  age: z.number().min(18, 'Must be at least 18').optional(),
  bio: z.string().max(500).optional(),
});

type UserFormData = z.infer<typeof userSchema>;
```

#### Conditional Validation

```tsx
const schema = z.discriminatedUnion('type', [
  z.object({
    type: z.literal('individual'),
    firstName: z.string().min(1),
    lastName: z.string().min(1),
  }),
  z.object({
    type: z.literal('company'),
    companyName: z.string().min(1),
    vatNumber: z.string().regex(/^[A-Z]{2}\d+$/),
  }),
]);
```

#### Nested Objects and Arrays

```tsx
const schema = z.object({
  name: z.string().min(1),
  addresses: z.array(z.object({
    street: z.string().min(1),
    city: z.string().min(1),
    zipCode: z.string().regex(/^\d{5}$/),
  })).min(1, 'At least one address required'),
  preferences: z.object({
    notifications: z.boolean(),
    language: z.enum(['en', 'it', 'fr']),
  }),
});
```

### Form Integration with MUI

#### Complete Form Example

```tsx
import { useForm, Controller } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { TextField, Button, MenuItem, FormControlLabel, Switch } from '@mui/material';

export function UserForm({ onSubmit, defaultValues }: UserFormProps) {
  const { control, handleSubmit, formState: { errors, isSubmitting } } = useForm<UserFormData>({
    resolver: zodResolver(userSchema),
    defaultValues,
  });

  return (
    <Box component="form" onSubmit={handleSubmit(onSubmit)} noValidate>
      <Controller
        name="name"
        control={control}
        render={({ field }) => (
          <TextField
            {...field}
            label="Name"
            error={!!errors.name}
            helperText={errors.name?.message}
            fullWidth
            required
          />
        )}
      />

      <Controller
        name="role"
        control={control}
        render={({ field }) => (
          <TextField
            {...field}
            select
            label="Role"
            error={!!errors.role}
            helperText={errors.role?.message}
            fullWidth
          >
            <MenuItem value="admin">Admin</MenuItem>
            <MenuItem value="user">User</MenuItem>
          </TextField>
        )}
      />

      <Controller
        name="active"
        control={control}
        render={({ field }) => (
          <FormControlLabel
            control={<Switch {...field} checked={field.value} />}
            label="Active"
          />
        )}
      />

      <Button type="submit" variant="contained" disabled={isSubmitting} fullWidth>
        {isSubmitting ? 'Saving...' : 'Save'}
      </Button>
    </Box>
  );
}
```

#### Form with Query Integration

```tsx
export function EditUserForm({ userId }: { userId: string }) {
  const { data: user, isLoading } = useQuery({
    queryKey: ['users', userId],
    queryFn: () => fetchUser(userId),
  });

  const mutation = useMutation({
    mutationFn: (data: UserFormData) => updateUser(userId, data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
      navigate('/users');
    },
  });

  if (isLoading) return <Skeleton />;

  return <UserForm defaultValues={user} onSubmit={mutation.mutateAsync} />;
}
```

### useFieldArray for Dynamic Fields

```tsx
import { useFieldArray } from 'react-hook-form';

const { fields, append, remove } = useFieldArray({ control, name: 'addresses' });

{fields.map((field, index) => (
  <Box key={field.id} sx={{ display: 'flex', gap: 2, mb: 2 }}>
    <Controller
      name={`addresses.${index}.street`}
      control={control}
      render={({ field }) => <TextField {...field} label="Street" />}
    />
    <IconButton onClick={() => remove(index)} aria-label="Remove address">
      <DeleteIcon />
    </IconButton>
  </Box>
))}
<Button onClick={() => append({ street: '', city: '', zipCode: '' })}>
  Add Address
</Button>
```

## React Router v7

### Route Configuration

```tsx
import { createBrowserRouter, RouterProvider, Navigate } from 'react-router-dom';

const router = createBrowserRouter([
  {
    path: '/login',
    element: <LoginPage />,
  },
  {
    path: '/',
    element: <ProtectedRoute><AdminLayout /></ProtectedRoute>,
    children: [
      { index: true, element: <Navigate to="/dashboard" replace /> },
      { path: 'dashboard', element: <DashboardPage /> },
      {
        path: 'users',
        children: [
          { index: true, element: <UsersIndexPage /> },
          { path: 'create', element: <UserCreatePage /> },
          { path: ':id', element: <UserShowPage /> },
          { path: ':id/edit', element: <UserEditPage /> },
        ],
      },
    ],
  },
  { path: '*', element: <NotFoundPage /> },
]);
```

### Protected Route Pattern

```tsx
import { Navigate, useLocation } from 'react-router-dom';
import { useAuth } from '@contexts/AuthContext';

function ProtectedRoute({ children }: { children: React.ReactNode }) {
  const { user, isLoading } = useAuth();
  const location = useLocation();

  if (isLoading) return <LoadingScreen />;
  if (!user) return <Navigate to="/login" state={{ from: location }} replace />;

  return <>{children}</>;
}
```

### Navigation Patterns

```tsx
import { useNavigate, useParams, useSearchParams, Link } from 'react-router-dom';

// Programmatic navigation
const navigate = useNavigate();
navigate('/users');
navigate(`/users/${id}/edit`);
navigate(-1); // Go back

// URL parameters
const { id } = useParams<{ id: string }>();

// Search params (for filters/pagination)
const [searchParams, setSearchParams] = useSearchParams();
const page = Number(searchParams.get('page') || '1');
setSearchParams({ page: String(page + 1), search: query });

// Declarative links
<Link to={`/users/${id}`}>View User</Link>
```

### Breadcrumbs with Router

```tsx
import { useMatches, Link } from 'react-router-dom';
import { Breadcrumbs, Typography } from '@mui/material';

function AppBreadcrumbs() {
  const matches = useMatches();
  const crumbs = matches.filter((m) => m.handle?.crumb);

  return (
    <Breadcrumbs aria-label="breadcrumb">
      {crumbs.map((match, index) => {
        const isLast = index === crumbs.length - 1;
        return isLast ? (
          <Typography key={match.id} color="text.primary">
            {match.handle.crumb(match.data)}
          </Typography>
        ) : (
          <Link key={match.id} to={match.pathname}>
            {match.handle.crumb(match.data)}
          </Link>
        );
      })}
    </Breadcrumbs>
  );
}
```

## State Management Strategy

### Decision Matrix

| State Type | Solution | Example |
|---|---|---|
| Server data | TanStack Query | API responses, user lists |
| Form state | React Hook Form | Input values, validation |
| Auth state | React Context | Current user, permissions |
| UI state (local) | useState | Drawer open, dialog visible |
| UI state (shared) | React Context or URL | Theme mode, filters |
| URL state | React Router | Pagination, active tab |

### Key Principles

1. **Server state** -> Always TanStack Query, never useState for API data
2. **Form state** -> Always React Hook Form + Zod, never manual validation
3. **Auth state** -> Single AuthContext at app root
4. **UI state** -> Keep as local as possible, lift only when necessary
5. **URL state** -> Use searchParams for filterable/shareable state

## Performance Patterns

### Memoization

> **React 19 Note**: The React 19 compiler can auto-memoize components, values, and callbacks. If using React 19 with the compiler enabled, manual `useMemo`, `useCallback`, and `memo` are often unnecessary. Use manual memoization only when targeting React 18 or when the compiler is not enabled.

```tsx
// Memoize expensive computations (React 18 or compiler not enabled)
const filteredUsers = useMemo(
  () => users.filter((u) => u.name.includes(search)),
  [users, search]
);

// Memoize callbacks passed to children
const handleDelete = useCallback(
  (id: string) => deleteMutation.mutate(id),
  [deleteMutation]
);

// Memoize components with stable props
const MemoizedCard = memo(UserCard);
```

### Lazy Loading Routes

```tsx
import { lazy, Suspense } from 'react';

const DashboardPage = lazy(() => import('@features/dashboard/pages/DashboardPage'));
const UsersPage = lazy(() => import('@features/users/pages/UsersIndexPage'));

// In router
{
  path: 'dashboard',
  element: (
    <Suspense fallback={<CircularProgress />}>
      <DashboardPage />
    </Suspense>
  ),
}
```

### Error Boundaries

```tsx
import { Component, type ReactNode } from 'react';

interface ErrorBoundaryState {
  hasError: boolean;
  error?: Error;
}

class ErrorBoundary extends Component<{ children: ReactNode; fallback?: ReactNode }, ErrorBoundaryState> {
  state: ErrorBoundaryState = { hasError: false };

  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback ?? (
        <Box sx={{ p: 4, textAlign: 'center' }}>
          <Typography variant="h5" gutterBottom>Something went wrong</Typography>
          <Button onClick={() => this.setState({ hasError: false })}>Try Again</Button>
        </Box>
      );
    }
    return this.props.children;
  }
}
```
