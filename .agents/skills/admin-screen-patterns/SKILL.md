---
name: admin-screen-patterns
description: Canonical high-density UI/UX layout patterns, screen archetypes, cockpit headers, metric strips, tab navigation, and sticky table architectures for the Karigarji Admin Panel.
license: Internal
metadata:
  version: "1.0.0"
  target: karigarji_admin_panel
---

# Karigarji Admin Screen Patterns & Design System

This skill codifies the authoritative, battle-tested UI/UX layout patterns for the **Karigarji Admin Panel** (`karigarji_admin_panel`). All pages, tabs, drawers, and tables MUST adhere to these archetypes to ensure uniform density, visual rhythm, responsive performance, and zero wasted screen real estate.

---

## 1. Core Principles (The Karigarji Standard)

1. **High-Density Desktop Ergonomics**:
   - The admin panel is a professional operational tool, not a marketing landing page.
   - Compact button heights: `h-7` (sub-actions/table buttons) to `h-8` (primary/toolbar buttons).
   - Typography scales: Primary titles `text-base font-bold`, section labels `text-xs font-semibold`, micro-labels `text-[10px] font-medium uppercase tracking-wider`, body/table text `text-xs`.
   - Tight padding: Containers use `p-3` or `p-3.5`, never bloated `p-6`.

2. **Content-Hugging Over Artificial Full-Width Stretching**:
   - **Never** stretch 3 or 4 metric cards across `grid grid-cols-1 sm:grid-cols-4 w-full` with `justify-between`. On 1440px+ widescreen displays, this leaves a gaping void between metric labels and icons.
   - Use **left-aligned, content-sized flex groups** (`flex items-center gap-2.5 flex-wrap shrink-0`). Each card hugs its content (~140px-180px width).

3. **Viewport-Anchored Internal Scrolling (Sticky Headers)**:
   - The shell (`AppLayout`) provides the boundary. The table or main workspace inside the page MUST own its own internal scrollbar.
   - Table headers must remain strictly pinned (`sticky top-0 z-20`) when scrolling through records.
   - Pagination bars must remain permanently pinned to the bottom of the card (`shrink-0 border-t bg-muted/20`).

---

## 2. Screen Archetype 1: Directory & List Workspace

Used for master registries (e.g., `CustomersPage`, `StaffUsersPage`, `FieldStaffPage`, `VendorsPage`).

### Layout Structure
```tsx
<PageContainer
  variant="fullBleed"
  className="h-full min-h-0 flex-1 flex flex-col p-3 sm:p-4 gap-2.5 overflow-hidden"
>
  {/* 1. Header Block */}
  <PageHeader
    sticky={false}
    title="Customers"
    description="Directory of registered consumers, commercial clients, and installed base accounts."
    className="pb-1 space-y-0.5 shrink-0"
    onRefresh={() => refetch()}
    isRefreshing={isLoading}
    actions={
      <Button onClick={handleCreate} size="sm" className="h-8 text-xs gap-1.5 cursor-pointer">
        <Plus className="size-3.5" />
        <span>Add Customer</span>
      </Button>
    }
  />

  {/* 2. Left-Aligned Content-Hugging Metric Cards */}
  <div className="flex items-center gap-2.5 shrink-0 flex-wrap">
    <Card className="shadow-xs border py-1.5 px-3 flex items-center gap-2.5 bg-card">
      <div className="size-7 rounded-md bg-primary/10 flex items-center justify-center text-primary shrink-0">
        <Users className="size-3.5" />
      </div>
      <div>
        <p className="text-[10px] font-medium text-muted-foreground uppercase tracking-wider">
          Total Customers
        </p>
        <p className="text-sm font-bold leading-none mt-0.5">{metrics.total}</p>
      </div>
    </Card>
    {/* Additional compact cards... */}
  </div>

  {/* 3. Filter Toolbar */}
  <div className="shrink-0">
    <CustomerFilterToolbar ... />
  </div>

  {/* 4. Viewport-Anchored Scrollable Table with Sticky Header */}
  <div className="flex-1 min-h-0 rounded-lg border border-border bg-card shadow-xs overflow-hidden flex flex-col">
    <div className="flex-1 min-h-0 overflow-auto">
      <table className="w-full caption-bottom text-sm border-collapse">
        <TableHeader className="sticky top-0 z-20 bg-muted [&_th]:sticky [&_th]:top-0 [&_th]:z-20 [&_th]:bg-muted [&_th]:shadow-[inset_0_-1px_0_var(--color-border)]">
          <TableRow className="hover:bg-transparent">
            <TableHead className="text-xs font-semibold py-2.5">Customer</TableHead>
            {/* Headers... */}
          </TableRow>
        </TableHeader>
        <TableBody>
          {/* Rows... */}
        </TableBody>
      </table>
    </div>

    {/* 5. Pinned Pagination Footer */}
    <div className="flex items-center justify-between text-xs text-muted-foreground px-3 py-2 border-t bg-muted/20 shrink-0">
      <p>Showing {items.length} of {total} records</p>
      <div className="flex items-center gap-1.5">
        {/* Pagination buttons (h-7 w-7) */}
      </div>
    </div>
  </div>
</PageContainer>
```

> [!IMPORTANT]
> **Sticky Header Anti-Pattern Alert**:
> Never use the default `<Table>` wrapper component inside `<div className="overflow-auto">`. The shadcn `<Table>` component automatically wraps `<table>` in an intermediate `<div data-slot="table-container" className="overflow-x-auto">` which breaks CSS `sticky top-0` vertical containment. Always use direct `<table className="w-full caption-bottom text-sm border-collapse">` inside the scroll div.

---

## 3. Screen Archetype 2: 360° Cockpit Detail View (Modern Stripe/Linear Pattern)

Used for comprehensive entity dashboards (e.g., `CustomerDetailPage`, `BranchStockItemPage`, `TechnicianStockPage`).

### Layout Structure
```tsx
<PageContainer variant="wide" className="space-y-4">
  {/* 1. Top Breadcrumb & Action Bar */}
  <div className="flex items-center justify-between gap-2 shrink-0 pt-1">
    <Button
      variant="ghost"
      size="sm"
      className="gap-1.5 text-xs text-muted-foreground hover:text-foreground h-7 px-2 cursor-pointer -ml-2"
      onClick={() => navigate("/accounts/customers")}
    >
      <ArrowLeft className="h-3.5 w-3.5" />
      <span>Customers</span>
    </Button>

    <div className="flex items-center gap-2">
      <Button variant="outline" size="sm" className="gap-1.5 h-8 text-xs font-medium px-3 cursor-pointer">
        <Edit className="size-3.5" />
        <span>Edit Customer</span>
      </Button>
      {/* Contextual actions */}
    </div>
  </div>

  {/* 2. Customer Hero Identity (Natural Page Flow, No Enclosing Card) */}
  <div className="flex flex-col sm:flex-row sm:items-center gap-4 py-1 shrink-0">
    <Avatar className="size-14 sm:size-16 rounded-2xl border border-border/80 shadow-xs shrink-0">
      <AvatarImage src={customer.avatarUrl} alt={customer.name} />
      <AvatarFallback className="bg-primary/10 text-primary font-bold text-xl rounded-2xl">
        {initials}
      </AvatarFallback>
    </Avatar>

    <div className="space-y-1.5 min-w-0">
      <div className="flex flex-wrap items-center gap-2.5">
        <h1 className="text-xl sm:text-2xl font-bold tracking-tight text-foreground truncate">
          {customer.name}
        </h1>
        <CustomerStatusBadge isActive={customer.isActive} isBlacklisted={customer.isBlacklisted} />
        {customer.gstin && (
          <Badge variant="outline" className="font-mono text-[10px] h-5 px-2 bg-muted/40 font-medium">
            B2B Commercial
          </Badge>
        )}
      </div>

      <div className="flex flex-wrap items-center gap-x-3 gap-y-1 text-xs text-muted-foreground">
        <span className="font-mono font-medium text-foreground flex items-center gap-1">
          <Phone className="size-3 text-muted-foreground" /> +91 {customer.phone}
        </span>
        <span className="text-muted-foreground/40">•</span>
        {customer.email && (
          <>
            <span className="flex items-center gap-1">
              <Mail className="size-3 text-muted-foreground" /> {customer.email}
            </span>
            <span className="text-muted-foreground/40">•</span>
          </>
        )}
        <span className="flex items-center gap-1">
          <Calendar className="size-3 text-muted-foreground" /> Joined {formattedDate}
        </span>
      </div>
    </div>
  </div>

  {/* 3. Unified Metric Ribbon (Single Sleek Band with Hairline Dividers) */}
  <div className="rounded-xl border border-border/70 bg-card divide-y sm:divide-y-0 sm:divide-x divide-border/60 grid grid-cols-2 sm:grid-cols-4 shadow-2xs shrink-0">
    <div className="p-3 sm:px-4 sm:py-3">
      <span className="text-xs text-muted-foreground block font-medium">Installed Machines</span>
      <span className="text-xl font-bold font-mono text-foreground mt-0.5 block">{appliancesCount}</span>
    </div>
    <div className="p-3 sm:px-4 sm:py-3">
      <span className="text-xs text-muted-foreground block font-medium">Service Locations</span>
      <span className="text-xl font-bold font-mono text-foreground mt-0.5 block">{addressesCount}</span>
    </div>
    <div className="p-3 sm:px-4 sm:py-3">
      <span className="text-xs text-muted-foreground block font-medium">Lifetime Bookings</span>
      <span className="text-xl font-bold font-mono text-foreground mt-0.5 block">{bookingsCount}</span>
    </div>
    <div className="p-3 sm:px-4 sm:py-3">
      <span className="text-xs text-muted-foreground block font-medium">Account Standing</span>
      <div className="flex items-center gap-2 mt-1">
        <span className="size-2 rounded-full bg-emerald-500" />
        <span className="text-xs font-semibold text-foreground">In Good Standing</span>
      </div>
    </div>
  </div>

  {/* 4. Sleek Underline Tabs (Linear / Vercel Pattern) */}
  <Tabs defaultValue="overview" className="space-y-4 pt-1">
    <TabsList className="bg-transparent border-b border-border w-full justify-start rounded-none h-auto p-0 gap-6 shrink-0 overflow-x-auto">
      <TabsTrigger
        value="overview"
        className="bg-transparent rounded-none border-b-2 border-transparent px-1 pb-2.5 pt-1 text-xs font-medium text-muted-foreground hover:text-foreground data-[state=active]:border-primary data-[state=active]:text-foreground data-[state=active]:shadow-none data-[state=active]:bg-transparent transition-colors cursor-pointer gap-2"
      >
        <MapPin className="size-3.5" />
        <span>Overview & Addresses</span>
        <span className="rounded-full px-1.5 py-0.2 text-[10px] bg-muted text-muted-foreground font-mono">
          {addressesCount}
        </span>
      </TabsTrigger>
      {/* Other tabs... */}
    </TabsList>

    {/* 5. Tab Content (Zero Margin Jump) */}
    <TabsContent value="overview" className="m-0 focus-visible:outline-hidden space-y-4">
      {/* Tab Workspaces */}
    </TabsContent>
  </Tabs>
</PageContainer>
```

---

## 4. Tab Workspace Layout Patterns

Inside any Cockpit Detail Tab (`TabsContent`), follow these specific micro-patterns:

### 1. High-Density KPI Micro-Strip (Inside Tabs)
- **Do NOT** use `grid grid-cols-4` stretching wide cards with `justify-between`.
- **DO** use left-aligned compact cards or an inline unified bar:
```tsx
<div className="flex items-center gap-2.5 shrink-0 flex-wrap">
  <div className="flex items-center gap-2.5 px-3 py-1.5 rounded-lg border border-border bg-card shadow-xs">
    <div className="size-6.5 rounded-md bg-primary/10 flex items-center justify-center text-primary shrink-0">
      <Wrench className="size-3.5" />
    </div>
    <div className="flex flex-col">
      <span className="text-[10px] font-medium text-muted-foreground uppercase tracking-wider">
        Installed Machines
      </span>
      <span className="text-sm font-bold leading-none mt-0.5">{appliancesCount}</span>
    </div>
  </div>
  {/* Additional metrics... */}
</div>
```

### 2. Tab Section Card & Table Viewports
- Section Headers: `p-3 border-b bg-muted/20 flex items-center justify-between`.
- Action buttons: `h-7 text-xs gap-1`.
- Grid Cards: When listing cards (e.g. Addresses, Appliances), use `grid grid-cols-1 md:grid-cols-2 xl:grid-cols-3 gap-2.5`.
- Keep individual cards compact (`p-3`, `text-xs`), with actions clearly visible in a bottom action row or upper-right dropdown.

---

## 5. Summary Cheat Sheet (CSS Tokens)

| Element | Standard Classes |
|---|---|
| **Primary Page Title** | `text-base font-bold tracking-tight text-foreground` |
| **Section Title** | `text-xs font-semibold text-foreground` |
| **Micro-Stat Label** | `text-[10px] font-medium text-muted-foreground uppercase tracking-wider` |
| **Stat Value** | `text-sm font-bold leading-none mt-0.5` |
| **Primary Toolbar Button** | `h-8 px-3 text-xs gap-1.5 font-medium` |
| **Sub-action / Table Button** | `h-7 px-2 text-xs gap-1 font-medium` |
| **Icon Container (Micro-card)**| `size-6.5` or `size-7 rounded-md flex items-center justify-center` |
| **Icon Inside Micro-card** | `size-3.5` |
| **Tab List Container** | `h-8 p-0.5 bg-muted/60 border border-border/40 rounded-lg` |
| **Tab Trigger** | `text-xs h-7 gap-1.5 px-2.5 data-[state=active]:bg-background` |
| **Counter Badge in Trigger** | `text-[10px] px-1 py-0 h-4` |
| **Sticky Table Header** | `sticky top-0 z-20 bg-muted [&_th]:sticky [&_th]:top-0 [&_th]:z-20 [&_th]:bg-muted [&_th]:shadow-[inset_0_-1px_0_var(--color-border)]` |
