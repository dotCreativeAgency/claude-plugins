---
name: mui-react-development
description: "This skill should be used when building, styling, or debugging React components with MUI (Material UI) v7, MUI X v8 (DataGrid, DatePicker), or related stack libraries. Activates for tasks like 'create a form', 'build a data table', 'add pagination', 'create a dialog', 'set up routing', 'add form validation', implementing server state with TanStack Query (useQuery, useMutation), configuring MUI themes, using the sx prop, setting up React Router routes, or working with any MUI component (Grid, Dialog, Drawer, TextField, etc.). Also relevant when the user mentions component styling, palette, typography, responsive layout, or Zod validation in React + MUI projects."
---

# MUI + React Full Stack Development Patterns

Best practices and patterns for building React applications with MUI (Material UI) v7, MUI X v8, TanStack Query v5, React Hook Form + Zod, and React Router v7.

## MUI v7 Critical Rules

### Grid Component (Most Common Mistake)

MUI v7 unified the Grid API. The old `Grid2` and `Unstable_Grid2` no longer exist.

```tsx
// CORRECT - MUI v7
import { Grid } from '@mui/material';

<Grid container spacing={2}>
  <Grid size={{ xs: 12, md: 6 }}>Content</Grid>
  <Grid size={{ xs: 12, md: 6 }}>Content</Grid>
</Grid>
```

**Never do:**
- Import `Grid2` or `Unstable_Grid2` (removed in v7)
- Use `item` prop (removed in v7)
- Use old breakpoint props: `<Grid xs={12} md={6}>` (use `size` prop instead)

### sx Prop Over Inline Styles

Always prefer `sx` prop for styling. It integrates with the theme and supports responsive values.

```tsx
// CORRECT
<Box sx={{ p: { xs: 2, md: 4 }, bgcolor: 'background.paper' }}>

// WRONG - never use inline styles
<Box style={{ padding: '16px', backgroundColor: '#fff' }}>
```

### Theme Tokens

Use theme values, never hardcode colors, spacing, or typography.

```tsx
// CORRECT - theme-aware
sx={{ color: 'primary.main', borderRadius: 1, p: 2 }}

// WRONG - hardcoded
sx={{ color: '#673ab7', borderRadius: '4px', padding: '16px' }}
```

### Slot Pattern

MUI v7 uses slots for component customization:

```tsx
<TextField
  slotProps={{
    input: { 'aria-label': 'Search' },
    inputLabel: { shrink: true }
  }}
/>
```

## MUI X v8 Patterns

### DataGrid

```tsx
import { DataGrid, type GridColDef, type GridPaginationModel } from '@mui/x-data-grid';
import { itIT } from '@mui/x-data-grid/locales'; // Italian locale

const columns: GridColDef[] = [
  { field: 'id', headerName: 'ID', width: 70 },
  { field: 'name', headerName: 'Name', flex: 1 },
];

<DataGrid
  rows={rows}
  columns={columns}
  paginationModel={paginationModel}
  onPaginationModelChange={setPaginationModel}
  pageSizeOptions={[10, 25, 50]}
  localeText={itIT.components.MuiDataGrid.defaultProps.localeText}
/>
```

### DatePickers

```tsx
import { DatePicker } from '@mui/x-date-pickers/DatePicker';
import { LocalizationProvider } from '@mui/x-date-pickers/LocalizationProvider';
import { AdapterDayjs } from '@mui/x-date-pickers/AdapterDayjs';

<LocalizationProvider dateAdapter={AdapterDayjs}>
  <DatePicker label="Select date" value={value} onChange={setValue} />
</LocalizationProvider>
```

## Component Architecture

### 150 LOC Rule

Every component file MUST stay under 150 lines. When approaching this limit, decompose:

```
ComponentName/
├── index.tsx              (5 LOC)  - Export only
├── ComponentName.tsx      (<100)   - Orchestration
├── ComponentNameHeader.tsx (<80)   - UI section
├── ComponentNameContent.tsx(<80)   - UI section
├── useComponentName.ts    (<80)    - Business logic hook
└── types.ts               (<50)   - Interfaces
```

### Component Template

```tsx
/**
 * @component ComponentName
 * @description Brief description of the component
 * @accessibility WCAG 2.1 AA compliant
 */

import { type FC } from 'react';
import { Box, Typography } from '@mui/material';

interface ComponentNameProps {
  /** Section title */
  title: string;
  /** Optional action callback */
  onAction?: () => void;
}

export const ComponentName: FC<ComponentNameProps> = ({ title, onAction }) => {
  return (
    <Box component="section" aria-label={title} sx={{ p: { xs: 2, sm: 3, md: 4 } }}>
      <Typography variant="h5" component="h2">{title}</Typography>
    </Box>
  );
};
```

## Accessibility Checklist (WCAG 2.1 AA)

Every component must include:
- `aria-label` or `aria-labelledby` on interactive containers
- Semantic HTML elements (`section`, `nav`, `main`, `header`)
- Heading hierarchy (h1 -> h2 -> h3, never skip levels)
- Visible focus indicators on interactive elements
- Complete keyboard navigation (Tab, Enter, Escape)
- Minimum touch target 44x44px
- Color contrast ratio >= 4.5:1

## TanStack Query v5

### useQuery for Data Fetching

```tsx
import { useQuery } from '@tanstack/react-query';

const { data, isLoading, error } = useQuery({
  queryKey: ['users', filters],
  queryFn: () => apiClient.get('/api/v1/users', { params: filters }),
  staleTime: 5 * 60 * 1000,
});
```

### useMutation for Data Changes

```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

const queryClient = useQueryClient();
const mutation = useMutation({
  mutationFn: (data: CreateUserData) => apiClient.post('/api/v1/users', data),
  onSuccess: () => queryClient.invalidateQueries({ queryKey: ['users'] }),
});
```

**Key rules:**
- Never use `useState` for server data - always use `useQuery`
- Invalidate related queries after mutations
- Use `queryKey` arrays for cache management
- Set appropriate `staleTime` based on data freshness needs

## React Hook Form + Zod

### Form with Validation

```tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  name: z.string().min(1, 'Required'),
  email: z.string().email('Invalid email'),
});

type FormData = z.infer<typeof schema>;

const { register, handleSubmit, formState: { errors } } = useForm<FormData>({
  resolver: zodResolver(schema),
});
```

### Integration with MUI

Use `register` for simple text inputs. Use `Controller` for MUI components that need `value`/`onChange` control (Select, Switch, DatePicker, Autocomplete).

```tsx
// Simple text input - register is sufficient
<TextField
  {...register('name')}
  error={!!errors.name}
  helperText={errors.name?.message}
  label="Name"
  fullWidth
/>

// Complex MUI component - use Controller
<Controller
  name="role"
  control={control}
  render={({ field }) => (
    <TextField {...field} select label="Role" error={!!errors.role}>
      <MenuItem value="admin">Admin</MenuItem>
      <MenuItem value="user">User</MenuItem>
    </TextField>
  )}
/>
```

## React Router v7

### Route Definition

```tsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom';

const router = createBrowserRouter([
  {
    path: '/',
    element: <Layout />,
    children: [
      { index: true, element: <Dashboard /> },
      { path: 'users', element: <Users /> },
      { path: 'users/:id', element: <UserDetail /> },
    ],
  },
]);
```

### Navigation and Params

```tsx
import { useNavigate, useParams } from 'react-router-dom';

const navigate = useNavigate();
const { id } = useParams<{ id: string }>();
```

## MUI MCP Server Setup

The **MUI MCP server** (`@mui/mcp`) provides direct access to official MUI documentation and API references. It should be installed and running for optimal MUI development support.

**Installation:**
```bash
npx -y @mui/mcp@latest
```

**MCP Configuration** (add to `.mcp.json` or project MCP settings):
```json
{
  "mcpServers": {
    "mui-mcp": {
      "command": "npx",
      "args": ["-y", "@mui/mcp@latest"]
    }
  }
}
```

If the MUI MCP server is unavailable, inform the user that documentation lookup is limited and provide the installation configuration above. Proceed using the patterns documented in this skill and reference files as fallback.

## MUI Documentation Lookup

### Step 1: Consult the MUI Documentation Index

Before writing MUI component code, fetch the MUI documentation index to identify the relevant documentation pages:

- **Documentation Index URL**: `https://mui.com/material-ui/llms.txt`
- Use `WebFetch` to retrieve the index and locate the specific component/API documentation page
- The index contains links to all MUI components, customization guides, migration guides, and integration docs
- If `WebFetch` is unavailable or the request fails, skip to Step 2 and use the MUI MCP tool directly

### Step 2: Verify API with MUI MCP

Use the documentation page URLs found in the index (Step 1) as input to the MUI MCP tool for precise API verification:

- `@mui/material` v7: Use `mcp__mui-mcp__useMuiDocs` with `@mui/material@7.2.0` docs URL
- `@mui/x-data-grid` v8: Use `@mui/x-data-grid@8.8.0` docs URL
- `@mui/x-date-pickers` v8: Use `@mui/x-date-pickers@8.8.0` docs URL

> **Note**: Version numbers above are verified at time of writing. Check for newer docs URLs if MUI has released major updates.

For non-MUI libraries in the stack (TanStack Query, Zod, React Router), use Context7 MCP: `mcp__plugin_context7_context7__resolve-library-id` + `query-docs`.

## Additional Resources

### Reference Files

For detailed patterns beyond the essentials above, consult these reference files:

- **`references/mui-v7-patterns.md`** - Consult when implementing Grid layouts, theme customization, responsive drawers, DataGrid with server-side pagination, custom column renderers, or Dialog/Card component patterns
- **`references/react-stack-patterns.md`** - Consult when implementing optimistic updates, dependent queries, complex form validation (conditional/nested/useFieldArray), protected routes, lazy loading, error boundaries, or state management decisions
