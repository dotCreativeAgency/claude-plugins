---
name: mui-react-development
description: "Use this skill for ANY React component work in MUI-based projects — building, styling, fixing, debugging, or refactoring. Covers MUI (Material UI) v7, MUI X v8 (DataGrid, DatePicker, Autocomplete), TanStack Query v5, React Hook Form + Zod, and React Router v7. Triggers on: 'create a form', 'build a data table', 'add server-side pagination', 'create a dialog', 'configure MUI theme', 'use the sx prop', 'add a snackbar', 'create tabs', 'build a page layout', 'fix the grid layout', 'the component doesn't align', useQuery, useMutation, or Italian: 'crea un componente React', 'aggiungi una tabella dati', 'crea una pagina', 'sistema il layout'. Also activates for Grid layout, responsive design, WCAG accessibility, component decomposition, and any task involving MUI imports or the sx prop — even if the user doesn't explicitly mention MUI."
---

# MUI + React Full Stack Development Patterns

Best practices and patterns for building React applications with MUI (Material UI) v7, MUI X v8, TanStack Query v5, React Hook Form + Zod, and React Router v7.

## MUI v7 Critical Rules

### Grid Component (Most Common Mistake)

MUI v7 unified the Grid API. The old `Grid2` and `Unstable_Grid2` no longer exist.

```tsx
// CORRECT - MUI v7
<Grid container spacing={2}>
  <Grid size={{ xs: 12, md: 6 }}>Content</Grid>
</Grid>

// WRONG - removed in v7
<Grid item xs={12} md={6}>Content</Grid>
<Grid2 size={{ xs: 12 }}>Content</Grid2>
```

> For responsive patterns, auto-sizing, and v6->v7 migration table, see `references/mui-v7-patterns.md`

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

MUI v7 replaced all capitalized `*Props` with the unified `slotProps` API. This applies across all MUI components, not just TextField.

```tsx
// CORRECT - MUI v7 slot pattern
<TextField
  slotProps={{
    input: { 'aria-label': 'Search' },
    inputLabel: { shrink: true }
  }}
/>

// WRONG - deprecated in MUI v7
<TextField
  InputProps={{ 'aria-label': 'Search' }}
  InputLabelProps={{ shrink: true }}
/>
```

```tsx
// Autocomplete — slotProps for listbox, paper, popper
<Autocomplete
  options={options}
  slotProps={{
    listbox: { sx: { maxHeight: 300 } },
    paper: { elevation: 4 },
    popper: { placement: 'bottom-start' }
  }}
  renderInput={(params) => <TextField {...params} label="Search" />}
/>

// Select — slotProps for the menu
<Select
  value={value}
  onChange={handleChange}
  slotProps={{
    input: { 'aria-label': 'Choose option' }
  }}
  MenuProps={{ PaperProps: { sx: { maxHeight: 300 } } }}
>
  <MenuItem value="a">Option A</MenuItem>
</Select>
```

## MUI X v8 Patterns

### DataGrid

```tsx
import { DataGrid, type GridColDef } from '@mui/x-data-grid';
import { itIT } from '@mui/x-data-grid/locales'; // FacileGestire uses Italian locale

<DataGrid
  rows={rows}
  columns={columns}
  paginationModel={paginationModel}
  onPaginationModelChange={setPaginationModel}
  pageSizeOptions={[10, 25, 50]}
  localeText={itIT.components.MuiDataGrid.defaultProps.localeText} // Adapt locale to project
/>
```

> For server-side pagination, custom column renderers, and toolbar patterns, see `references/mui-v7-patterns.md`

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

### 150 LOC Rule (Project Convention)

> **Note**: This is a FacileGestire project convention, not a universal MUI rule. Check the target project's CLAUDE.md for its own component size limits.

Every component file stays under 150 lines. When approaching this limit, decompose into a folder structure:

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
import { type FC } from 'react';
import { Box, Typography } from '@mui/material';

interface ComponentNameProps {
  title: string;
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

Key points: semantic HTML (`component="section"`), `aria-label` on containers, responsive `sx` values, typed props interface.

## Accessibility Checklist (WCAG 2.1 AA)

Every component must include:
- `aria-label` or `aria-labelledby` on interactive containers
- Semantic HTML elements (`section`, `nav`, `main`, `header`)
- Heading hierarchy (h1 -> h2 -> h3, never skip levels)
- Visible focus indicators on interactive elements
- Complete keyboard navigation (Tab, Enter, Escape)
- Minimum touch target 44x44px
- Color contrast ratio >= 4.5:1

## React 19 Considerations

React 19 introduces the React Compiler which auto-memoizes components, values, and callbacks. If the target project uses React 19 with the compiler enabled, manual `useMemo`, `useCallback`, and `memo` are unnecessary — the compiler handles it. Only add manual memoization when targeting React 18 or when the compiler is not enabled.

## Common MUI Patterns

### Snackbar / Toast Notifications

Use `notistack` (already integrated in FacileGestire projects) or MUI's Snackbar for user feedback after CRUD operations:

```tsx
import { useSnackbar } from 'notistack';

const { enqueueSnackbar } = useSnackbar();

// After mutation success
onSuccess: () => {
  enqueueSnackbar('Record saved successfully', { variant: 'success' });
  queryClient.invalidateQueries({ queryKey: ['items'] });
},
onError: () => {
  enqueueSnackbar('Failed to save record', { variant: 'error' });
},
```

### Autocomplete with Controller

Autocomplete requires `Controller` because it manages complex value/onChange behavior:

```tsx
<Controller
  name="supplier"
  control={control}
  render={({ field: { onChange, value } }) => (
    <Autocomplete
      options={suppliers}
      getOptionLabel={(opt) => opt.name}
      value={suppliers.find((s) => s.id === value) ?? null}
      onChange={(_, newValue) => onChange(newValue?.id ?? null)}
      renderInput={(params) => (
        <TextField {...params} label="Supplier" error={!!errors.supplier} helperText={errors.supplier?.message} />
      )}
    />
  )}
/>
```

### Tabs

```tsx
import { Tabs, Tab, Box } from '@mui/material';

const [tabIndex, setTabIndex] = useState(0);

<Tabs value={tabIndex} onChange={(_, v) => setTabIndex(v)} aria-label="Section tabs">
  <Tab label="Details" id="tab-0" aria-controls="tabpanel-0" />
  <Tab label="History" id="tab-1" aria-controls="tabpanel-1" />
</Tabs>
<Box role="tabpanel" id="tabpanel-0" hidden={tabIndex !== 0} aria-labelledby="tab-0">
  {tabIndex === 0 && <DetailsContent />}
</Box>
<Box role="tabpanel" id="tabpanel-1" hidden={tabIndex !== 1} aria-labelledby="tab-1">
  {tabIndex === 1 && <HistoryContent />}
</Box>
```

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

> For error handling, optimistic updates, dependent queries, and prefetching patterns, see `references/react-stack-patterns.md`

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

Use the documentation page URLs found in the index (Step 1) as input to `mcp__mui-mcp__useMuiDocs` for precise API verification.

**Important**: Read the target project's `package.json` to determine exact MUI package versions before querying docs. Use the installed version (e.g., `@mui/material@7.x.x`) in the docs URL, not hardcoded version numbers, to ensure documentation matches the project.

For non-MUI libraries in the stack (TanStack Query, Zod, React Router), use Context7 MCP: `mcp__plugin_context7_context7__resolve-library-id` + `query-docs`.

## Additional Resources

### Reference Files

For detailed patterns beyond the essentials above, consult these reference files:

- **`references/mui-v7-patterns.md`** - Consult when implementing Grid layouts, theme customization, dark mode, responsive drawers, DataGrid with server-side pagination, custom column renderers, toolbar patterns, or Dialog/Card/Loading component patterns
- **`references/react-stack-patterns.md`** - Consult when implementing optimistic updates, dependent queries, prefetching, complex form validation (conditional/nested/useFieldArray), protected routes, breadcrumbs, lazy loading, error boundaries, or state management decisions. File includes a table of contents for quick navigation
