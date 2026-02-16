---
name: page-header-generator
description: "This skill should be used when the user asks to 'create a page', 'build a page', 'add a new page', 'crea una pagina', 'aggiungi pagina', 'page header', 'create header', 'crea header', 'header component', or when creating any new Index, Show, Create, or Edit page. Also triggers when building page layouts, implementing list pages, detail pages, form pages, or any route-level component that needs a header. Provides theme-driven design system, shared header components, and page structure standards."
---

# Page Header Generator & Page Creation Standard

Standard for creating pages in React + MUI applications. Defines header types,
shared components, theme-driven design system, and page structure conventions.

## Theme-Driven Design Principle

All header colors MUST be derived from the application's MUI theme -- never hardcode color values. Before creating any header, read the project's theme configuration to understand the primary and secondary palette.

```tsx
// CORRECT - derive from theme
background: (theme) => `linear-gradient(135deg, ${theme.palette.primary.dark} 0%, ${theme.palette.primary.light} 100%)`
border: (theme) => `1px solid ${alpha(theme.palette.primary.dark, 0.3)}`
boxShadow: (theme) => `0 4px 12px ${alpha(theme.palette.primary.dark, 0.3)}`

// WRONG - hardcoded colors
background: 'linear-gradient(135deg, #00695c 0%, #26a69a 100%)'
```

## Shared Header Components

Three reusable header components live in `src/components/headers/`. Import from `@/components/headers`.

### IndexPageHeader (Index/List pages)

Light primary gradient. For list pages with optional count chip and action button.

```tsx
import { IndexPageHeader } from '@/components/headers';

<IndexPageHeader
  title="Entities"
  subtitle="Manage all entities"
  totalCount={rowCount}           // optional - shows primary-colored count chip
  actionLabel="New Entity"        // optional - shows solid primary action button
  actionIcon={<AddIcon />}        // optional - button start icon
  onAction={() => navigate('/entities/create')}  // required if actionLabel set
/>
```

### ShowPageHeader (Detail/Show pages)

Dark primary gradient with glassmorphism. For entity detail pages with status and actions.

```tsx
import { ShowPageHeader } from '@/components/headers';

<ShowPageHeader
  title={entity.name}
  icon={<Business sx={{ fontSize: 32 }} />}  // 32px icon inside 64x64 glassmorphism box
  isActive={entity.status === 'active'}       // green/red glassmorphism status chip
  backPath="/entities"
  backLabel="Back to entity list"
  canEdit={entity.status !== 'trashed'}       // optional, default true
  canDelete={false}                           // optional, default false
  onEdit={() => navigate(`/entities/${id}/edit`)}
  onDelete={() => handleDelete()}             // optional
/>
```

### EditPageHeader (Edit pages)

Dark primary gradient with glassmorphism. For entity edit pages with back button and status.

```tsx
import { EditPageHeader } from '@/components/headers';

<EditPageHeader
  entityName={entity.name}                    // renders "Edit {entityName}"
  icon={<Business />}                         // 24px icon inside 48x48 glassmorphism box
  subtitle="Edit entity information"
  isActive={entity.status === 'active'}       // optional - shows status chip
  onBack={() => navigate('/entities')}
/>
```

### Create Headers (Per-Entity)

Create headers remain **per-entity components** (not shared) because each has unique title,
subtitle, icon, and back path hardcoded. Follow the Create template in
[references/templates.md](references/templates.md).

## Page Structure Standards

### Page Type Reference

| Type | Header Component | Container | Content Area |
|------|-----------------|-----------|--------------|
| **Index** | `<IndexPageHeader>` | None (Box section) | Paper with DataGrid |
| **Show** | `<ShowPageHeader>` | `Container maxWidth="lg"` | Detail sections |
| **Create** | Per-entity component | `Container maxWidth="md"` or `"lg"` | Paper with form |
| **Edit** | `<EditPageHeader>` | `Container maxWidth="md"` or `"lg"` | Paper with form |

### Standard Page Layout

```tsx
// Index page pattern
<Box component="section" aria-label="Entity management">
  <IndexPageHeader ... />
  <Paper elevation={0} sx={{ p: 3, border: 1, borderColor: 'divider', borderRadius: 3 }}>
    {/* DataGrid or content */}
  </Paper>
</Box>

// Show page pattern
<Container maxWidth="lg" sx={{ py: 3 }}>
  <ShowPageHeader ... />
  <DetailSections ... />
</Container>

// Create page pattern
<Container maxWidth="md" sx={{ py: 3 }}>
  <EntityCreatePageHeader />
  <Paper elevation={0} sx={{ p: 4, border: 1, borderColor: 'divider', borderRadius: 3 }}>
    <Form ... />
  </Paper>
</Container>

// Edit page pattern
<Container maxWidth="lg" sx={{ py: 3 }}>
  <EditPageHeader ... />
  <Paper elevation={1} sx={{ p: 3, borderRadius: 2, border: '1px solid', borderColor: 'divider' }}>
    <Form ... />
  </Paper>
</Container>
```

### Loading & Error States

Every Show/Edit page must handle loading and error states before rendering:

```tsx
if (isLoading) {
  return (
    <Box sx={{ display: 'grid', placeItems: 'center', minHeight: '40vh' }}>
      <CircularProgress aria-label="Loading entity" />
    </Box>
  );
}

if (isError || !data) {
  return (
    <Container maxWidth="lg" sx={{ py: 3 }}>
      <Alert severity="error">Entity not found or error loading data.</Alert>
    </Container>
  );
}
```

## Theme-Driven Design System

All colors are derived from the application's MUI theme. The table below shows the pattern, not specific hex values.

| Property | Dark Gradient (Create/Edit/Show) | Light Gradient (Index) |
|----------|----------------------------------|------------------------|
| Gradient | `primary.dark -> primary.light` | `primary[50] -> primary[100]` (or light alpha variants) |
| Text | `#ffffff` (contrast text) | Standard MUI text |
| Button | Glassmorphism (translucent white) | Solid `primary.main` / hover `primary.dark` |
| Container | `Box` borderRadius `12px` | `Paper elevation={0}` borderRadius `3` |
| Border | `alpha(primary.dark, 0.3)` | `borderColor: 'divider'` |
| BoxShadow | `0 4px 12px alpha(primary.dark, 0.3)` | None |
| Icon box (Show) | 64x64, borderRadius `14px` | N/A |
| Icon box (Create/Edit) | 48x48, borderRadius `10px` | N/A |
| Status chip | Glassmorphism green/red (success/error palette) | N/A |

### Glassmorphism Pattern

```tsx
// Shared pattern for buttons, icon containers, chips on dark gradient
backgroundColor: 'rgba(255, 255, 255, 0.15)',
border: '1px solid rgba(255, 255, 255, 0.2)',
backdropFilter: 'blur(10px)',
'&:hover': { backgroundColor: 'rgba(255, 255, 255, 0.25)' }
```

### Theme Color Access Pattern

```tsx
import { alpha } from '@mui/material/styles';

// Dark gradient header container
sx={(theme) => ({
  background: `linear-gradient(135deg, ${theme.palette.primary.dark} 0%, ${theme.palette.primary.light} 100%)`,
  border: `1px solid ${alpha(theme.palette.primary.dark, 0.3)}`,
  boxShadow: `0 4px 12px ${alpha(theme.palette.primary.dark, 0.3)}`,
})}

// Light gradient header container (Index pages)
sx={(theme) => ({
  background: `linear-gradient(135deg, ${alpha(theme.palette.primary.main, 0.08)} 0%, ${alpha(theme.palette.primary.main, 0.15)} 100%)`,
})}

// Solid primary action button
sx={(theme) => ({
  backgroundColor: theme.palette.primary.main,
  '&:hover': { backgroundColor: theme.palette.primary.dark },
})}

// Count chip
sx={(theme) => ({
  backgroundColor: theme.palette.primary.main,
  color: theme.palette.primary.contrastText,
  fontWeight: 700,
})}

// Status chip - active
sx={(theme) => ({
  backgroundColor: alpha(theme.palette.success.main, 0.2),
  border: `1px solid ${alpha(theme.palette.success.main, 0.4)}`,
  color: '#ffffff',
})}

// Status chip - inactive
sx={(theme) => ({
  backgroundColor: alpha(theme.palette.error.main, 0.2),
  border: `1px solid ${alpha(theme.palette.error.main, 0.4)}`,
  color: '#ffffff',
})}
```

## Common Icons by Domain

```tsx
// Import from @mui/icons-material
Business, Domain          // Companies, Organizations
AdminPanelSettings        // Administrators
PersonOutline, PersonAdd  // Users
Home, HomeWork            // Properties
Settings                  // Configuration
Assignment                // Logs, Documents
Security                  // Security
Notifications             // Notifications
Dashboard                 // Dashboard
CreditCard                // Payments
Analytics                 // Reports
```

## When to Create Custom vs Use Shared

**Use shared components** (default):
- Standard Index, Show, or Edit pages
- Consistent design system across all pages

**Create per-entity component** (exception):
- Create headers (unique title/subtitle/icon per entity)
- Headers with entity-specific features not covered by shared props
- If additional custom sections needed inside the header

Custom per-entity headers follow the Create template pattern:
`memo` wrapping, `displayName`, placed in `features/[module]/components/`.

## Templates Reference

Full Create header template in [references/templates.md](references/templates.md).
