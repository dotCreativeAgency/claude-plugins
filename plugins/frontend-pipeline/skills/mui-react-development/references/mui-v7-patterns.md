# MUI v7 Comprehensive Patterns

## Grid System (v7 Unified API)

### Basic Usage

MUI v7 removed `Grid2` and `Unstable_Grid2`. The `Grid` component IS the new API.

```tsx
import { Grid } from '@mui/material';

// Container with spacing
<Grid container spacing={2}>
  <Grid size={{ xs: 12, sm: 6, md: 4 }}>
    <Card>Item 1</Card>
  </Grid>
  <Grid size={{ xs: 12, sm: 6, md: 4 }}>
    <Card>Item 2</Card>
  </Grid>
  <Grid size={{ xs: 12, sm: 6, md: 4 }}>
    <Card>Item 3</Card>
  </Grid>
</Grid>
```

### Responsive Patterns

```tsx
// Full-width on mobile, split on desktop
<Grid container spacing={{ xs: 1, sm: 2, md: 3 }}>
  <Grid size={{ xs: 12, md: 8 }}>Main content</Grid>
  <Grid size={{ xs: 12, md: 4 }}>Sidebar</Grid>
</Grid>

// Auto-sizing
<Grid container spacing={2}>
  <Grid size="auto">Fixed width content</Grid>
  <Grid size="grow">Takes remaining space</Grid>
</Grid>
```

### Common Migration Mistakes

| Old API (v6) | New API (v7) |
|---|---|
| `<Grid item xs={6}>` | `<Grid size={{ xs: 6 }}>` |
| `<Grid item>` | `<Grid>` (no item prop needed) |
| `import Grid2` | `import { Grid } from '@mui/material'` |
| `xs={12} md={6}` | `size={{ xs: 12, md: 6 }}` |

## Theme Configuration

### Creating a Custom Theme

```tsx
import { createTheme, ThemeProvider } from '@mui/material/styles';

const theme = createTheme({
  palette: {
    primary: { main: '#673ab7' },
    secondary: { main: '#ff5722' },
  },
  typography: {
    fontFamily: '"Inter", "Roboto", "Helvetica", "Arial", sans-serif',
    button: { textTransform: 'none' },
  },
  shape: { borderRadius: 12 },
  components: {
    MuiButton: {
      defaultProps: { variant: 'contained', disableElevation: true },
      styleOverrides: {
        root: { borderRadius: 8 },
      },
    },
    MuiCard: {
      styleOverrides: {
        root: { borderRadius: 12 },
      },
    },
    MuiPaper: {
      styleOverrides: {
        root: { borderRadius: 12 },
      },
    },
  },
});
```

### Dark Mode Support

```tsx
const theme = createTheme({
  palette: {
    mode: prefersDarkMode ? 'dark' : 'light',
    primary: { main: '#673ab7' },
  },
});
```

### Accessing Theme in Components

```tsx
// Via sx prop (preferred)
<Box sx={{ color: 'primary.main', bgcolor: 'background.paper' }} />

// Via useTheme hook
import { useTheme } from '@mui/material/styles';
const theme = useTheme();

// Via styled
import { styled } from '@mui/material/styles';
const StyledBox = styled(Box)(({ theme }) => ({
  padding: theme.spacing(2),
  color: theme.palette.primary.main,
}));
```

## Component Patterns

### Dialog Pattern

```tsx
import { Dialog, DialogTitle, DialogContent, DialogActions, Button } from '@mui/material';

interface ConfirmDialogProps {
  open: boolean;
  title: string;
  message: string;
  onConfirm: () => void;
  onCancel: () => void;
}

export const ConfirmDialog: FC<ConfirmDialogProps> = ({
  open, title, message, onConfirm, onCancel
}) => (
  <Dialog
    open={open}
    onClose={onCancel}
    aria-labelledby="confirm-dialog-title"
    maxWidth="sm"
    fullWidth
  >
    <DialogTitle id="confirm-dialog-title">{title}</DialogTitle>
    <DialogContent>
      <Typography>{message}</Typography>
    </DialogContent>
    <DialogActions>
      <Button onClick={onCancel}>Cancel</Button>
      <Button onClick={onConfirm} variant="contained" color="error">
        Confirm
      </Button>
    </DialogActions>
  </Dialog>
);
```

### Responsive Drawer / Sidebar

```tsx
import { Drawer, useMediaQuery, useTheme } from '@mui/material';

const theme = useTheme();
const isMobile = useMediaQuery(theme.breakpoints.down('md'));

<Drawer
  variant={isMobile ? 'temporary' : 'permanent'}
  open={isMobile ? drawerOpen : true}
  onClose={() => setDrawerOpen(false)}
  sx={{
    width: 240,
    '& .MuiDrawer-paper': { width: 240 },
  }}
>
  {drawerContent}
</Drawer>
```

### Card with Actions

```tsx
<Card sx={{ borderRadius: 2 }}>
  <CardContent>
    <Typography variant="h6" component="h3" gutterBottom>
      {title}
    </Typography>
    <Typography variant="body2" color="text.secondary">
      {description}
    </Typography>
  </CardContent>
  <CardActions sx={{ justifyContent: 'flex-end', px: 2, pb: 2 }}>
    <Button size="small" onClick={onEdit}>Edit</Button>
    <Button size="small" color="error" onClick={onDelete}>Delete</Button>
  </CardActions>
</Card>
```

### Loading States

```tsx
import { Skeleton, CircularProgress, LinearProgress } from '@mui/material';

// Skeleton for content placeholders
<Skeleton variant="text" width={200} />
<Skeleton variant="rectangular" height={200} />
<Skeleton variant="circular" width={40} height={40} />

// Full-page loading
<Box sx={{ display: 'flex', justifyContent: 'center', alignItems: 'center', minHeight: '60vh' }}>
  <CircularProgress />
</Box>

// Top-of-page loading bar
{isLoading && <LinearProgress sx={{ position: 'fixed', top: 0, left: 0, right: 0, zIndex: 9999 }} />}
```

## Responsive Design

### Breakpoint Usage

```tsx
// In sx prop
sx={{
  display: { xs: 'none', md: 'block' },
  flexDirection: { xs: 'column', sm: 'row' },
  p: { xs: 1, sm: 2, md: 3 },
  fontSize: { xs: '0.875rem', md: '1rem' },
}}

// With useMediaQuery
const isMobile = useMediaQuery(theme.breakpoints.down('sm'));
const isTablet = useMediaQuery(theme.breakpoints.between('sm', 'md'));
const isDesktop = useMediaQuery(theme.breakpoints.up('md'));
```

### Breakpoint Values

| Breakpoint | Min Width |
|---|---|
| xs | 0px |
| sm | 600px |
| md | 900px |
| lg | 1200px |
| xl | 1536px |

## DataGrid Advanced Patterns

### Server-Side Pagination

```tsx
const [paginationModel, setPaginationModel] = useState<GridPaginationModel>({
  page: 0,
  pageSize: 25,
});

const { data, isLoading } = useQuery({
  queryKey: ['items', paginationModel],
  queryFn: () => fetchItems(paginationModel.page + 1, paginationModel.pageSize),
});

<DataGrid
  rows={data?.data ?? []}
  columns={columns}
  rowCount={data?.total ?? 0}
  loading={isLoading}
  paginationModel={paginationModel}
  onPaginationModelChange={setPaginationModel}
  paginationMode="server"
  pageSizeOptions={[10, 25, 50]}
  localeText={itIT.components.MuiDataGrid.defaultProps.localeText}
/>
```

### Custom Column Renderers

```tsx
const columns: GridColDef[] = [
  {
    field: 'status',
    headerName: 'Status',
    width: 120,
    renderCell: (params) => (
      <Chip
        label={params.value}
        color={params.value === 'active' ? 'success' : 'default'}
        size="small"
      />
    ),
  },
  {
    field: 'actions',
    headerName: 'Actions',
    type: 'actions',
    width: 100,
    getActions: (params) => [
      <GridActionsCellItem icon={<EditIcon />} label="Edit" onClick={() => onEdit(params.id)} />,
      <GridActionsCellItem icon={<DeleteIcon />} label="Delete" onClick={() => onDelete(params.id)} />,
    ],
  },
];
```

### Toolbar with Search and Export

```tsx
import { GridToolbarContainer, GridToolbarExport, GridToolbarFilterButton } from '@mui/x-data-grid';

function CustomToolbar() {
  return (
    <GridToolbarContainer>
      <GridToolbarFilterButton />
      <GridToolbarExport />
    </GridToolbarContainer>
  );
}

<DataGrid slots={{ toolbar: CustomToolbar }} />
```

## Common Pitfalls

1. **Using `Grid2` import** - Does not exist in MUI v7, use `Grid`
2. **Hardcoding colors** - Always use theme tokens via sx prop
3. **Inline styles** - Use sx prop instead of `style` attribute
4. **Missing aria-labels** - Every interactive element needs accessible labeling
5. **Forgetting locale** - DataGrid needs Italian locale for i18n projects
6. **Using `InputProps`** - MUI v7 uses `slotProps` instead of `InputProps`/`inputProps`
7. **Missing responsive values** - Always design mobile-first with ascending breakpoints
