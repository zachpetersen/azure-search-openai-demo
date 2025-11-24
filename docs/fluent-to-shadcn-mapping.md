# Fluent UI to shadcn/ui Component Migration Mapping

## Overview

This document provides a detailed mapping from Fluent UI components (v8 and v9) to shadcn/ui equivalents for migrating the azure-search-openai-demo frontend to align with px.ai.ops.dash standards.

**Target Framework**: shadcn/ui (Radix UI + Tailwind CSS)  
**Source Frameworks**: @fluentui/react (v8), @fluentui/react-components (v9)

## General Migration Principles

### Styling Approach

**Fluent UI:**
```tsx
// Inline styles and theme-based styling
<Stack tokens={{ childrenGap: 10 }} styles={{ root: { padding: 20 } }}>
```

**shadcn/ui:**
```tsx
// Tailwind utility classes
<div className="flex flex-col gap-2.5 p-5">
```

### Component Composition

**Fluent UI:**
- Components often handle multiple concerns (layout, styling, behavior)
- Heavy use of props for configuration

**shadcn/ui:**
- Smaller, composable primitives
- Composition over configuration
- More explicit JSX structure

## Component Mapping Reference

### 1. Buttons

#### DefaultButton (Fluent v8) → Button (shadcn)

**Fluent UI:**
```tsx
import { DefaultButton } from "@fluentui/react";

<DefaultButton 
  text="Click me"
  onClick={handleClick}
  disabled={isDisabled}
  primary={false}
/>
```

**shadcn/ui:**
```tsx
import { Button } from "@/components/ui/button";

<Button 
  onClick={handleClick}
  disabled={isDisabled}
  variant="outline"  // or "default", "secondary", "ghost", "link"
>
  Click me
</Button>
```

**Prop Mapping:**
| Fluent Prop | shadcn/ui Equivalent | Notes |
|------------|---------------------|-------|
| `text` | children | Pass as child content |
| `primary={true}` | `variant="default"` | Primary action |
| `primary={false}` | `variant="outline"` | Secondary action |
| `disabled` | `disabled` | Same |
| `onClick` | `onClick` | Same |
| `ariaLabel` | `aria-label` | Same |
| `icon` | See Icons section | Compose with icon component |

#### PrimaryButton (Fluent v8) → Button (shadcn)

**Fluent UI:**
```tsx
import { PrimaryButton } from "@fluentui/react";

<PrimaryButton text="Submit" onClick={handleSubmit} />
```

**shadcn/ui:**
```tsx
import { Button } from "@/components/ui/button";

<Button onClick={handleSubmit}>Submit</Button>
// variant="default" is the default (primary styled)
```

#### IconButton (Fluent v8) → Button with Icon (shadcn)

**Fluent UI:**
```tsx
import { IconButton } from "@fluentui/react";

<IconButton 
  iconProps={{ iconName: 'Delete' }}
  ariaLabel="Delete"
  onClick={handleDelete}
/>
```

**shadcn/ui:**
```tsx
import { Button } from "@/components/ui/button";
import { Trash2 } from "lucide-react";

<Button 
  variant="ghost" 
  size="icon"
  aria-label="Delete"
  onClick={handleDelete}
>
  <Trash2 className="h-4 w-4" />
</Button>
```

**Icon Library Change:**
- Fluent: `@fluentui/react-icons`
- shadcn: `lucide-react` (recommended) or custom SVGs

#### Button (Fluent v9) → Button (shadcn)

**Fluent UI v9:**
```tsx
import { Button } from "@fluentui/react-components";

<Button appearance="primary" onClick={onClick}>
  Click me
</Button>
```

**shadcn/ui:**
```tsx
import { Button } from "@/components/ui/button";

<Button onClick={onClick}>Click me</Button>
```

**Appearance Mapping:**
| Fluent v9 | shadcn/ui |
|-----------|-----------|
| `appearance="primary"` | `variant="default"` |
| `appearance="secondary"` | `variant="secondary"` |
| `appearance="outline"` | `variant="outline"` |
| `appearance="subtle"` | `variant="ghost"` |
| `appearance="transparent"` | `variant="link"` |

### 2. Form Controls

#### TextField → Input

**Fluent UI:**
```tsx
import { TextField } from "@fluentui/react";

<TextField
  label="Username"
  value={username}
  onChange={(e, newValue) => setUsername(newValue || '')}
  placeholder="Enter username"
  required
  errorMessage={error}
  description="Your unique identifier"
/>
```

**shadcn/ui:**
```tsx
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";

<div className="space-y-2">
  <Label htmlFor="username">
    Username <span className="text-red-500">*</span>
  </Label>
  <Input
    id="username"
    value={username}
    onChange={(e) => setUsername(e.target.value)}
    placeholder="Enter username"
    required
  />
  {error && <p className="text-sm text-red-500">{error}</p>}
  <p className="text-sm text-muted-foreground">Your unique identifier</p>
</div>
```

**Key Differences:**
- shadcn requires explicit Label component
- onChange receives standard React event (not Fluent's transformed value)
- Error messages and descriptions are separate elements
- More verbose but more flexible

**Reusable Form Field Component:**
```tsx
// Create a reusable FormField component
interface FormFieldProps {
  label: string;
  error?: string;
  description?: string;
  required?: boolean;
  children: React.ReactNode;
}

function FormField({ label, error, description, required, children }: FormFieldProps) {
  return (
    <div className="space-y-2">
      <Label>
        {label}
        {required && <span className="text-red-500 ml-1">*</span>}
      </Label>
      {children}
      {error && <p className="text-sm text-red-500">{error}</p>}
      {description && <p className="text-sm text-muted-foreground">{description}</p>}
    </div>
  );
}

// Usage:
<FormField label="Username" required error={error} description="Your unique identifier">
  <Input 
    value={username} 
    onChange={(e) => setUsername(e.target.value)}
    placeholder="Enter username"
  />
</FormField>
```

#### Checkbox → Checkbox

**Fluent UI:**
```tsx
import { Checkbox } from "@fluentui/react";

<Checkbox
  label="Remember me"
  checked={checked}
  onChange={(e, isChecked) => setChecked(!!isChecked)}
/>
```

**shadcn/ui:**
```tsx
import { Checkbox } from "@/components/ui/checkbox";
import { Label } from "@/components/ui/label";

<div className="flex items-center space-x-2">
  <Checkbox
    id="remember"
    checked={checked}
    onCheckedChange={setChecked}
  />
  <Label htmlFor="remember">Remember me</Label>
</div>
```

**Key Differences:**
- Label is separate component
- Uses `onCheckedChange` instead of `onChange`
- `onCheckedChange` receives boolean directly (not event)

#### Dropdown → Select

**Fluent UI:**
```tsx
import { Dropdown, IDropdownOption } from "@fluentui/react";

const options: IDropdownOption[] = [
  { key: 'apple', text: 'Apple' },
  { key: 'banana', text: 'Banana' },
  { key: 'orange', text: 'Orange' }
];

<Dropdown
  label="Choose a fruit"
  selectedKey={selectedFruit}
  onChange={(e, option) => setSelectedFruit(option?.key as string)}
  options={options}
/>
```

**shadcn/ui:**
```tsx
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";
import { Label } from "@/components/ui/label";

<div className="space-y-2">
  <Label htmlFor="fruit">Choose a fruit</Label>
  <Select value={selectedFruit} onValueChange={setSelectedFruit}>
    <SelectTrigger id="fruit">
      <SelectValue placeholder="Select a fruit" />
    </SelectTrigger>
    <SelectContent>
      <SelectItem value="apple">Apple</SelectItem>
      <SelectItem value="banana">Banana</SelectItem>
      <SelectItem value="orange">Orange</SelectItem>
    </SelectContent>
  </Select>
</div>
```

**Key Differences:**
- More compositional (Trigger, Content, Item)
- Values are strings, not objects
- `onValueChange` receives value directly
- Requires explicit placeholder

### 3. Layout Components

#### Stack → Tailwind Flex/Grid

**Fluent UI:**
```tsx
import { Stack } from "@fluentui/react";

// Vertical stack
<Stack tokens={{ childrenGap: 10 }} styles={{ root: { padding: 20 } }}>
  <div>Item 1</div>
  <div>Item 2</div>
  <div>Item 3</div>
</Stack>

// Horizontal stack
<Stack horizontal tokens={{ childrenGap: 8 }}>
  <Button>Cancel</Button>
  <Button primary>Save</Button>
</Stack>
```

**shadcn/ui (Tailwind):**
```tsx
// Vertical stack
<div className="flex flex-col gap-2.5 p-5">
  <div>Item 1</div>
  <div>Item 2</div>
  <div>Item 3</div>
</div>

// Horizontal stack
<div className="flex flex-row gap-2">
  <Button variant="outline">Cancel</Button>
  <Button>Save</Button>
</div>
```

**Gap Mapping:**
| Fluent tokens.childrenGap | Tailwind gap-* |
|--------------------------|----------------|
| 4 | `gap-1` (0.25rem) |
| 8 | `gap-2` (0.5rem) |
| 10 | `gap-2.5` (0.625rem) |
| 12 | `gap-3` (0.75rem) |
| 16 | `gap-4` (1rem) |
| 20 | `gap-5` (1.25rem) |
| 24 | `gap-6` (1.5rem) |

### 4. Overlays & Dialogs

#### Panel → Sheet / Dialog

**Fluent UI:**
```tsx
import { Panel, PanelType } from "@fluentui/react";

<Panel
  isOpen={isOpen}
  onDismiss={onClose}
  headerText="Settings"
  type={PanelType.medium}
  closeButtonAriaLabel="Close"
>
  <div>Panel content here</div>
</Panel>
```

**shadcn/ui (Sheet - for side panels):**
```tsx
import { Sheet, SheetContent, SheetDescription, SheetHeader, SheetTitle } from "@/components/ui/sheet";

<Sheet open={isOpen} onOpenChange={setIsOpen}>
  <SheetContent>
    <SheetHeader>
      <SheetTitle>Settings</SheetTitle>
      <SheetDescription>Configure your preferences</SheetDescription>
    </SheetHeader>
    <div className="py-4">
      Panel content here
    </div>
  </SheetContent>
</Sheet>
```

**shadcn/ui (Dialog - for modals):**
```tsx
import { Dialog, DialogContent, DialogDescription, DialogHeader, DialogTitle } from "@/components/ui/dialog";

<Dialog open={isOpen} onOpenChange={setIsOpen}>
  <DialogContent>
    <DialogHeader>
      <DialogTitle>Settings</DialogTitle>
      <DialogDescription>Configure your preferences</DialogDescription>
    </DialogHeader>
    <div>
      Dialog content here
    </div>
  </DialogContent>
</Dialog>
```

**Panel Type Mapping:**
| Fluent PanelType | shadcn Sheet side + className |
|-----------------|------------------------------|
| `PanelType.small` | `side="right" className="w-80"` |
| `PanelType.medium` | `side="right" className="w-96"` |
| `PanelType.large` | `side="right" className="w-1/2"` |
| `PanelType.extraLarge` | `side="right" className="w-2/3"` |
| `PanelType.custom(600)` | `side="right" className="w-[600px]"` |

#### Callout → Popover

**Fluent UI:**
```tsx
import { Callout } from "@fluentui/react";

<Callout
  target={`#${buttonId}`}
  onDismiss={onDismiss}
  role="dialog"
  gapSpace={0}
>
  <div style={{ padding: 20 }}>
    Callout content
  </div>
</Callout>
```

**shadcn/ui:**
```tsx
import { Popover, PopoverContent, PopoverTrigger } from "@/components/ui/popover";

<Popover open={isOpen} onOpenChange={setIsOpen}>
  <PopoverTrigger asChild>
    <Button>Open popover</Button>
  </PopoverTrigger>
  <PopoverContent className="w-80">
    <div className="space-y-2">
      Popover content
    </div>
  </PopoverContent>
</Popover>
```

**Key Differences:**
- shadcn Popover uses trigger-based pattern (not target selector)
- If you need positioning relative to an element, wrap that element in PopoverTrigger
- For tooltips specifically, use Tooltip component instead

### 5. Feedback Components

#### Spinner → Spinner (custom) or Loading state

**Fluent UI:**
```tsx
import { Spinner, SpinnerSize } from "@fluentui/react";

<Spinner size={SpinnerSize.large} label="Loading..." />
```

**shadcn/ui:**
```tsx
// Option 1: Create a custom Spinner component
import { Loader2 } from "lucide-react";

<div className="flex flex-col items-center gap-2">
  <Loader2 className="h-8 w-8 animate-spin text-primary" />
  <p className="text-sm text-muted-foreground">Loading...</p>
</div>

// Option 2: Use Button loading state
import { Button } from "@/components/ui/button";
import { Loader2 } from "lucide-react";

<Button disabled>
  <Loader2 className="mr-2 h-4 w-4 animate-spin" />
  Loading...
</Button>
```

**Size Mapping:**
| Fluent SpinnerSize | Lucide icon size |
|-------------------|------------------|
| `SpinnerSize.xSmall` | `h-3 w-3` |
| `SpinnerSize.small` | `h-4 w-4` |
| `SpinnerSize.medium` | `h-6 w-6` |
| `SpinnerSize.large` | `h-8 w-8` |

**Custom Spinner Component:**
```tsx
// components/ui/spinner.tsx
import { Loader2 } from "lucide-react";
import { cn } from "@/lib/utils";

interface SpinnerProps {
  size?: "sm" | "md" | "lg";
  className?: string;
}

const sizeMap = {
  sm: "h-4 w-4",
  md: "h-6 w-6",
  lg: "h-8 w-8"
};

export function Spinner({ size = "md", className }: SpinnerProps) {
  return (
    <Loader2 
      className={cn("animate-spin text-primary", sizeMap[size], className)} 
    />
  );
}
```

#### MessageBar → Alert

**Fluent UI:**
```tsx
import { MessageBar, MessageBarType } from "@fluentui/react";

<MessageBar
  messageBarType={MessageBarType.error}
  isMultiline={false}
  onDismiss={onDismiss}
>
  An error occurred while processing your request.
</MessageBar>
```

**shadcn/ui:**
```tsx
import { Alert, AlertDescription, AlertTitle } from "@/components/ui/alert";
import { AlertCircle, X } from "lucide-react";
import { Button } from "@/components/ui/button";

<Alert variant="destructive" className="relative">
  <AlertCircle className="h-4 w-4" />
  <AlertTitle>Error</AlertTitle>
  <AlertDescription>
    An error occurred while processing your request.
  </AlertDescription>
  <Button
    variant="ghost"
    size="icon"
    className="absolute right-2 top-2"
    onClick={onDismiss}
  >
    <X className="h-4 w-4" />
  </Button>
</Alert>
```

**MessageBarType Mapping:**
| Fluent MessageBarType | shadcn Alert variant + icon |
|----------------------|---------------------------|
| `MessageBarType.info` | `variant="default"` + Info icon |
| `MessageBarType.success` | `variant="default"` + CheckCircle icon |
| `MessageBarType.warning` | `variant="default"` + AlertTriangle icon |
| `MessageBarType.error` | `variant="destructive"` + AlertCircle icon |

### 6. Navigation Components

#### Pivot/PivotItem → Tabs

**Fluent UI:**
```tsx
import { Pivot, PivotItem } from "@fluentui/react";

<Pivot
  selectedKey={selectedTab}
  onLinkClick={(item) => setSelectedTab(item?.props.itemKey || '')}
>
  <PivotItem headerText="Overview" itemKey="overview">
    <div>Overview content</div>
  </PivotItem>
  <PivotItem headerText="Details" itemKey="details">
    <div>Details content</div>
  </PivotItem>
  <PivotItem headerText="Settings" itemKey="settings">
    <div>Settings content</div>
  </PivotItem>
</Pivot>
```

**shadcn/ui:**
```tsx
import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs";

<Tabs value={selectedTab} onValueChange={setSelectedTab}>
  <TabsList>
    <TabsTrigger value="overview">Overview</TabsTrigger>
    <TabsTrigger value="details">Details</TabsTrigger>
    <TabsTrigger value="settings">Settings</TabsTrigger>
  </TabsList>
  <TabsContent value="overview">
    <div>Overview content</div>
  </TabsContent>
  <TabsContent value="details">
    <div>Details content</div>
  </TabsContent>
  <TabsContent value="settings">
    <div>Settings content</div>
  </TabsContent>
</Tabs>
```

### 7. Data Display

#### Link → Link (HTML + Tailwind)

**Fluent UI:**
```tsx
import { Link } from "@fluentui/react";

<Link href="https://example.com" target="_blank">
  Visit our website
</Link>
```

**shadcn/ui:**
```tsx
// Use standard <a> tag with Tailwind classes
<a 
  href="https://example.com" 
  target="_blank"
  rel="noopener noreferrer"
  className="text-primary underline-offset-4 hover:underline"
>
  Visit our website
</a>

// Or create a reusable Link component
import { cn } from "@/lib/utils";

interface LinkProps extends React.AnchorHTMLAttributes<HTMLAnchorElement> {
  children: React.ReactNode;
}

export function Link({ className, children, ...props }: LinkProps) {
  return (
    <a
      className={cn(
        "text-primary underline-offset-4 hover:underline",
        className
      )}
      {...props}
    >
      {children}
    </a>
  );
}
```

#### Label → Label

**Fluent UI:**
```tsx
import { Label } from "@fluentui/react";

<Label>Email address</Label>
```

**shadcn/ui:**
```tsx
import { Label } from "@/components/ui/label";

<Label htmlFor="email">Email address</Label>
```

**Key Difference:**
- shadcn Label should be associated with an input via `htmlFor`

#### Text → HTML + Tailwind

**Fluent UI:**
```tsx
import { Text } from "@fluentui/react";

<Text variant="large" block>Large text block</Text>
<Text variant="small">Small text</Text>
```

**shadcn/ui:**
```tsx
// Use semantic HTML with Tailwind typography
<p className="text-lg">Large text block</p>
<p className="text-sm">Small text</p>

// For consistent typography, define classes
<p className="text-base">Base text</p>
<p className="text-sm text-muted-foreground">Helper text</p>
<h2 className="text-2xl font-bold">Heading</h2>
```

**Text Variant Mapping:**
| Fluent Text variant | Tailwind classes |
|--------------------|------------------|
| `variant="tiny"` | `text-xs` |
| `variant="xSmall"` | `text-xs` |
| `variant="small"` | `text-sm` |
| `variant="medium"` | `text-base` |
| `variant="large"` | `text-lg` |
| `variant="xLarge"` | `text-xl` |
| `variant="xxLarge"` | `text-2xl` |

### 8. Icons

#### Fluent Icons → Lucide React

**Fluent UI:**
```tsx
import { 
  Delete24Regular,
  Add24Regular,
  Settings24Regular,
  Sparkle28Filled
} from "@fluentui/react-icons";

<Delete24Regular />
<Add24Regular />
```

**shadcn/ui (Lucide React):**
```tsx
import { 
  Trash2,
  Plus,
  Settings,
  Sparkles
} from "lucide-react";

<Trash2 className="h-6 w-6" />
<Plus className="h-6 w-6" />
```

**Icon Mapping Table:**
| Fluent Icon | Lucide React | Notes |
|------------|--------------|-------|
| `Delete24Regular` | `Trash2` | Trash can |
| `Add24Regular` | `Plus` | Plus sign |
| `Settings24Regular` | `Settings` | Gear icon |
| `Sparkle28Filled` | `Sparkles` | AI/magic icon |
| `History24Regular` | `History` | Clock arrow |
| `Mic28Filled` | `Mic` | Microphone |
| `Send28Filled` | `Send` | Send arrow |
| `ErrorCircle24Regular` | `AlertCircle` | Error/alert |
| `LocalLanguage24Regular` | `Languages` | Language/globe |

**Icon Size Mapping:**
| Fluent Size Suffix | Lucide className |
|-------------------|------------------|
| `16` | `h-4 w-4` |
| `20` | `h-5 w-5` |
| `24` | `h-6 w-6` |
| `28` | `h-7 w-7` |
| `32` | `h-8 w-8` |

### 9. Utility Hooks

#### useId → React.useId

**Fluent UI:**
```tsx
import { useId } from "@fluentui/react-hooks";

function MyComponent() {
  const inputId = useId('input');
  return (
    <>
      <label htmlFor={inputId}>Name</label>
      <input id={inputId} />
    </>
  );
}
```

**React 18+ (Built-in):**
```tsx
import { useId } from "react";

function MyComponent() {
  const inputId = useId();
  return (
    <>
      <label htmlFor={inputId}>Name</label>
      <input id={inputId} />
    </>
  );
}
```

**Key Differences:**
- React's `useId` doesn't take a prefix parameter
- React's `useId` returns a string like `:r1:`
- For readable IDs, you can still compose: `const inputId = \`input-${useId()}\``

#### useBoolean → useState

**Fluent UI:**
```tsx
import { useBoolean } from "@fluentui/react-hooks";

function MyComponent() {
  const [isOpen, { setTrue: openPanel, setFalse: closePanel, toggle }] = useBoolean(false);
  
  return (
    <button onClick={openPanel}>Open</button>
  );
}
```

**React (useState):**
```tsx
import { useState, useCallback } from "react";

function MyComponent() {
  const [isOpen, setIsOpen] = useState(false);
  const openPanel = useCallback(() => setIsOpen(true), []);
  const closePanel = useCallback(() => setIsOpen(false), []);
  const toggle = useCallback(() => setIsOpen(prev => !prev), []);
  
  return (
    <button onClick={openPanel}>Open</button>
  );
}

// Or simpler without callbacks:
function MyComponent() {
  const [isOpen, setIsOpen] = useState(false);
  
  return (
    <button onClick={() => setIsOpen(true)}>Open</button>
  );
}
```

### 10. Tooltips

#### Tooltip (Fluent v9) → Tooltip (shadcn)

**Fluent UI:**
```tsx
import { Tooltip, Button } from "@fluentui/react-components";

<Tooltip content="Click to save" relationship="label">
  <Button>Save</Button>
</Tooltip>
```

**shadcn/ui:**
```tsx
import { Tooltip, TooltipContent, TooltipProvider, TooltipTrigger } from "@/components/ui/tooltip";
import { Button } from "@/components/ui/button";

<TooltipProvider>
  <Tooltip>
    <TooltipTrigger asChild>
      <Button>Save</Button>
    </TooltipTrigger>
    <TooltipContent>
      <p>Click to save</p>
    </TooltipContent>
  </Tooltip>
</TooltipProvider>
```

**Note:** Wrap your app root with `<TooltipProvider>` once, then you don't need it for each tooltip:

```tsx
// App.tsx
import { TooltipProvider } from "@/components/ui/tooltip";

function App() {
  return (
    <TooltipProvider>
      {/* Your app content */}
    </TooltipProvider>
  );
}
```

## Migration Patterns

### Pattern 1: Form with Multiple Fields

**Before (Fluent):**
```tsx
function SettingsForm() {
  return (
    <Stack tokens={{ childrenGap: 16 }}>
      <TextField
        label="Name"
        value={name}
        onChange={(e, v) => setName(v || '')}
      />
      <Dropdown
        label="Role"
        selectedKey={role}
        options={roleOptions}
        onChange={(e, option) => setRole(option?.key as string)}
      />
      <Checkbox
        label="Enable notifications"
        checked={notifications}
        onChange={(e, checked) => setNotifications(!!checked)}
      />
      <Stack horizontal tokens={{ childrenGap: 8 }}>
        <DefaultButton text="Cancel" onClick={onCancel} />
        <PrimaryButton text="Save" onClick={onSave} />
      </Stack>
    </Stack>
  );
}
```

**After (shadcn):**
```tsx
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";
import { Checkbox } from "@/components/ui/checkbox";
import { Button } from "@/components/ui/button";

function SettingsForm() {
  return (
    <div className="space-y-4">
      <div className="space-y-2">
        <Label htmlFor="name">Name</Label>
        <Input
          id="name"
          value={name}
          onChange={(e) => setName(e.target.value)}
        />
      </div>
      
      <div className="space-y-2">
        <Label htmlFor="role">Role</Label>
        <Select value={role} onValueChange={setRole}>
          <SelectTrigger id="role">
            <SelectValue placeholder="Select role" />
          </SelectTrigger>
          <SelectContent>
            {roleOptions.map(option => (
              <SelectItem key={option.key} value={option.key}>
                {option.text}
              </SelectItem>
            ))}
          </SelectContent>
        </Select>
      </div>
      
      <div className="flex items-center space-x-2">
        <Checkbox
          id="notifications"
          checked={notifications}
          onCheckedChange={setNotifications}
        />
        <Label htmlFor="notifications">Enable notifications</Label>
      </div>
      
      <div className="flex gap-2">
        <Button variant="outline" onClick={onCancel}>Cancel</Button>
        <Button onClick={onSave}>Save</Button>
      </div>
    </div>
  );
}
```

### Pattern 2: Panel with Settings

**Before (Fluent):**
```tsx
import { Panel, PanelType } from "@fluentui/react";

<Panel
  isOpen={isOpen}
  onDismiss={onClose}
  headerText="Advanced Settings"
  type={PanelType.medium}
>
  <Stack tokens={{ childrenGap: 20 }}>
    {/* Settings content */}
  </Stack>
</Panel>
```

**After (shadcn):**
```tsx
import { Sheet, SheetContent, SheetDescription, SheetHeader, SheetTitle } from "@/components/ui/sheet";

<Sheet open={isOpen} onOpenChange={setIsOpen}>
  <SheetContent className="w-96">
    <SheetHeader>
      <SheetTitle>Advanced Settings</SheetTitle>
      <SheetDescription>Configure advanced options</SheetDescription>
    </SheetHeader>
    <div className="space-y-5 py-4">
      {/* Settings content */}
    </div>
  </SheetContent>
</Sheet>
```

### Pattern 3: Button with Icon and Tooltip

**Before (Fluent):**
```tsx
import { IconButton } from "@fluentui/react";
import { Delete24Regular } from "@fluentui/react-icons";
import { Tooltip } from "@fluentui/react-components";

<Tooltip content="Delete item" relationship="label">
  <IconButton
    iconProps={{ iconName: 'Delete' }}
    onClick={handleDelete}
  />
</Tooltip>
```

**After (shadcn):**
```tsx
import { Button } from "@/components/ui/button";
import { Tooltip, TooltipContent, TooltipTrigger } from "@/components/ui/tooltip";
import { Trash2 } from "lucide-react";

<Tooltip>
  <TooltipTrigger asChild>
    <Button
      variant="ghost"
      size="icon"
      onClick={handleDelete}
    >
      <Trash2 className="h-4 w-4" />
    </Button>
  </TooltipTrigger>
  <TooltipContent>
    <p>Delete item</p>
  </TooltipContent>
</Tooltip>
```

## Common Pitfalls & Solutions

### 1. Event Handler Signatures Changed

**Problem:**
```tsx
// Fluent - event handlers receive transformed values
onChange={(e, newValue) => setValue(newValue || '')}
```

**Solution:**
```tsx
// shadcn - standard React events
onChange={(e) => setValue(e.target.value)}
```

### 2. Missing Labels for Form Fields

**Problem:**
```tsx
// Fluent - label is a prop
<TextField label="Name" />
```

**Solution:**
```tsx
// shadcn - explicit Label component
<Label htmlFor="name">Name</Label>
<Input id="name" />
```

### 3. Stack Replaced with Tailwind

**Problem:**
```tsx
// Fluent - complex Stack configuration
<Stack tokens={{ childrenGap: 10, padding: 20 }} horizontal>
```

**Solution:**
```tsx
// shadcn - Tailwind flex utilities
<div className="flex flex-row gap-2.5 p-5">
```

### 4. Icon Sizes

**Problem:**
```tsx
// Fluent - size in icon name
<Delete24Regular />
```

**Solution:**
```tsx
// shadcn - size via className
<Trash2 className="h-6 w-6" />
```

### 5. Panel/Dialog Open State

**Problem:**
```tsx
// Fluent - isOpen prop
<Panel isOpen={isOpen} onDismiss={onClose} />
```

**Solution:**
```tsx
// shadcn - open/onOpenChange pattern
<Sheet open={isOpen} onOpenChange={setIsOpen}>
```

## Testing After Migration

### Visual Regression Testing

- Take screenshots of all Fluent UI components
- Compare with shadcn/ui versions
- Ensure visual parity (spacing, colors, sizes)

### Functional Testing Checklist

- [ ] All form inputs maintain controlled/uncontrolled behavior
- [ ] Validation and error messages display correctly
- [ ] Dropdowns/selects work with keyboard navigation
- [ ] Dialogs/sheets open and close correctly
- [ ] Buttons trigger correct actions
- [ ] Tooltips appear on hover
- [ ] Icons are correct size and color
- [ ] Accessibility attributes preserved (aria-label, role, etc.)
- [ ] Tab order is logical
- [ ] Screen reader compatibility maintained

### Performance

- Check bundle size before and after migration
- Fluent UI is typically heavier than shadcn/ui
- Expected reduction: 50-200KB (minified + gzipped)

## Migration Checklist by Component

Track migration progress:

### Buttons
- [ ] DefaultButton → Button (outline variant)
- [ ] PrimaryButton → Button (default variant)
- [ ] IconButton → Button (ghost + icon variant)
- [ ] Button (v9) → Button

### Forms
- [ ] TextField → Input + Label
- [ ] Checkbox → Checkbox + Label
- [ ] Dropdown → Select

### Layout
- [ ] Stack (vertical) → div with flex flex-col
- [ ] Stack (horizontal) → div with flex flex-row

### Overlays
- [ ] Panel → Sheet
- [ ] Callout → Popover
- [ ] Dialog → Dialog

### Feedback
- [ ] Spinner → Custom Spinner or Loader2
- [ ] MessageBar → Alert

### Navigation
- [ ] Pivot/PivotItem → Tabs

### Data Display
- [ ] Link → HTML anchor with Tailwind
- [ ] Label → Label
- [ ] Text → HTML elements with Tailwind

### Icons
- [ ] All @fluentui/react-icons → lucide-react equivalents

### Utilities
- [ ] useId → React.useId
- [ ] useBoolean → useState

## Additional Resources

### shadcn/ui Documentation
- Main docs: https://ui.shadcn.com
- Components: https://ui.shadcn.com/docs/components
- Theming: https://ui.shadcn.com/docs/theming

### Tailwind CSS
- Flex utilities: https://tailwindcss.com/docs/flex
- Spacing: https://tailwindcss.com/docs/padding
- Typography: https://tailwindcss.com/docs/font-size

### Lucide Icons
- Icon search: https://lucide.dev/icons

### Radix UI (shadcn's foundation)
- Primitives: https://www.radix-ui.com/primitives/docs/overview/introduction
- Accessibility: https://www.radix-ui.com/primitives/docs/overview/accessibility

---

**Document Version**: 1.0  
**Last Updated**: 2025-11-24  
**Prepared for**: px.ai.ops.dash Integration
