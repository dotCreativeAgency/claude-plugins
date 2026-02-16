# Page Header Templates & References

## Create Page Header Template

Create headers are per-entity components (not shared) because each has unique hardcoded
title, subtitle, icon, and back path. Use `memo` wrapping and `displayName`.

### Placeholders

| Placeholder | Description | Example |
|-------------|-------------|---------|
| `{{EntityName}}` | PascalCase entity name | `Supplier` |
| `{{TITLE}}` | Page title | `New Supplier` |
| `{{SUBTITLE}}` | Page subtitle | `Enter information for the new supplier` |
| `{{BACK_BUTTON}}` | Back button aria-label | `Back to supplier list` |
| `{{IconName}}` | MUI icon name | `AddBusiness` |
| `{{backPath}}` | Back navigation path | `/suppliers` |

### Template

```tsx
import { memo } from 'react';
import { Box, Typography, IconButton } from '@mui/material';
import { alpha } from '@mui/material/styles';
import { ArrowBack, {{IconName}} } from '@mui/icons-material';
import { useNavigate } from 'react-router-dom';

const {{EntityName}}CreatePageHeaderComponent = () => {
  const navigate = useNavigate();

  return (
    <Box
      sx={(theme) => ({
        mb: 3,
        borderRadius: '12px',
        background: `linear-gradient(135deg, ${theme.palette.primary.dark} 0%, ${theme.palette.primary.light} 100%)`,
        border: `1px solid ${alpha(theme.palette.primary.dark, 0.3)}`,
        boxShadow: `0 4px 12px ${alpha(theme.palette.primary.dark, 0.3)}`,
        overflow: 'hidden',
      })}
    >
      <Box
        sx={{
          p: 3,
          display: 'flex',
          alignItems: 'center',
          gap: 3,
        }}
      >
        {/* Glassmorphism Back Button */}
        <IconButton
          onClick={() => navigate('{{backPath}}')}
          aria-label="{{BACK_BUTTON}}"
          sx={{
            color: '#ffffff',
            backgroundColor: 'rgba(255, 255, 255, 0.15)',
            border: '1px solid rgba(255, 255, 255, 0.2)',
            backdropFilter: 'blur(10px)',
            '&:hover': { backgroundColor: 'rgba(255, 255, 255, 0.25)' },
          }}
        >
          <ArrowBack />
        </IconButton>

        {/* Icon Container - Glassmorphism */}
        <Box
          sx={{
            width: 48,
            height: 48,
            borderRadius: '10px',
            backgroundColor: 'rgba(255, 255, 255, 0.15)',
            border: '1px solid rgba(255, 255, 255, 0.2)',
            display: 'flex',
            alignItems: 'center',
            justifyContent: 'center',
            backdropFilter: 'blur(10px)',
          }}
        >
          <{{IconName}} sx={{ fontSize: 24, color: '#ffffff' }} />
        </Box>

        {/* Text Information */}
        <Box sx={{ flex: 1 }}>
          <Typography
            variant="h5"
            sx={{
              fontWeight: 600,
              color: '#ffffff',
              letterSpacing: '-0.01em',
              mb: 0.5,
            }}
          >
            {{TITLE}}
          </Typography>
          <Typography
            variant="body2"
            sx={{ color: 'rgba(255, 255, 255, 0.9)' }}
          >
            {{SUBTITLE}}
          </Typography>
        </Box>
      </Box>
    </Box>
  );
};

export const {{EntityName}}CreatePageHeader = memo({{EntityName}}CreatePageHeaderComponent);
{{EntityName}}CreatePageHeader.displayName = '{{EntityName}}CreatePageHeader';
```

---

## Shared Component Props Reference

### IndexPageHeaderProps

```typescript
interface IndexPageHeaderProps {
  title: string;           // Page title (h4, fontWeight 600)
  subtitle: string;        // Page subtitle (body1, text.secondary)
  totalCount?: number;     // Shows primary-colored count chip next to title
  actionLabel?: string;    // Shows solid primary action button
  actionIcon?: ReactNode;  // Button start icon
  onAction?: () => void;   // Button click handler (required if actionLabel set)
}
```

### ShowPageHeaderProps

```typescript
interface ShowPageHeaderProps {
  title: string;           // Entity name (h5, white, fontWeight 600)
  icon: ReactNode;         // 32px icon inside 64x64 glassmorphism box
  isActive: boolean;       // Green/red glassmorphism status chip
  backPath: string;        // Back navigation path
  backLabel: string;       // Back button aria-label
  canEdit?: boolean;       // Show edit button (default: true)
  canDelete?: boolean;     // Show delete button (default: false)
  onEdit?: () => void;     // Edit button handler
  onDelete?: () => void;   // Delete button handler
}
```

### EditPageHeaderProps

```typescript
interface EditPageHeaderProps {
  entityName: string;      // Renders "Edit {entityName}" (h5, white)
  icon: ReactNode;         // 24px icon inside 48x48 glassmorphism box
  subtitle: string;        // Subtitle (body2, white 0.9 opacity)
  isActive?: boolean;      // Optional glassmorphism status chip
  onBack: () => void;      // Back button handler
}
```

---

## Theme-Driven Design System Constants

All colors derive from the application's MUI theme. Never hardcode color values.

```tsx
import { alpha } from '@mui/material/styles';

// Dark gradient container (Create/Edit/Show pages)
sx={(theme) => ({
  borderRadius: '12px',
  background: `linear-gradient(135deg, ${theme.palette.primary.dark} 0%, ${theme.palette.primary.light} 100%)`,
  border: `1px solid ${alpha(theme.palette.primary.dark, 0.3)}`,
  boxShadow: `0 4px 12px ${alpha(theme.palette.primary.dark, 0.3)}`,
})}

// Light gradient container (Index pages)
sx={(theme) => ({
  background: `linear-gradient(135deg, ${alpha(theme.palette.primary.main, 0.08)} 0%, ${alpha(theme.palette.primary.main, 0.15)} 100%)`,
  border: 1,
  borderColor: 'divider',
  borderRadius: 3,
})}
// Uses Paper elevation={0}

// Glassmorphism pattern (buttons, icons, chips on dark gradient)
backgroundColor: 'rgba(255, 255, 255, 0.15)',
border: '1px solid rgba(255, 255, 255, 0.2)',
backdropFilter: 'blur(10px)',
'&:hover': { backgroundColor: 'rgba(255, 255, 255, 0.25)' }

// Solid primary action button (on light gradient)
sx={(theme) => ({
  backgroundColor: theme.palette.primary.main,
  '&:hover': { backgroundColor: theme.palette.primary.dark },
})}

// Count chip (on light gradient)
sx={(theme) => ({
  backgroundColor: theme.palette.primary.main,
  color: theme.palette.primary.contrastText,
  fontWeight: 700,
})}

// Status chip - active (glassmorphism)
sx={(theme) => ({
  backgroundColor: alpha(theme.palette.success.main, 0.2),
  border: `1px solid ${alpha(theme.palette.success.main, 0.4)}`,
  color: '#ffffff',
})}

// Status chip - inactive (glassmorphism)
sx={(theme) => ({
  backgroundColor: alpha(theme.palette.error.main, 0.2),
  border: `1px solid ${alpha(theme.palette.error.main, 0.4)}`,
  color: '#ffffff',
})}
```
