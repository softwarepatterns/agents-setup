---
name: shadcn
description: Manages shadcn components and projects — adding, searching, fixing, debugging, styling, and composing UI. Provides project context, component docs, and usage examples. Applies when working with shadcn/ui, component registries, presets, or any project with a components.json file.
user-invocable: false
---

# shadcn/ui

A framework for building ui, components and design systems. Components are added as source code to the user's project via the CLI.

> **IMPORTANT:** Run all CLI commands using the project's package runner: `npx shadcn@latest`, `pnpm dlx shadcn@latest`, or `bunx --bun shadcn@latest` — based on the project's `packageManager`. Examples below use `npx shadcn@latest`.

## Current Project Context

Run `npx shadcn@latest info --json` for the project config and installed components. Use `npx shadcn@latest docs <component>` to get documentation and example URLs for any component.

## Principles

1. **Use existing components first.** `npx shadcn@latest search` before writing custom UI.
2. **Compose, don't reinvent.** Settings = Tabs + Card + forms. Dashboard = Sidebar + Card + Chart + Table.
3. **Use built-in variants before custom styles.** `variant="outline"`, `size="sm"`, etc.
4. **Use semantic colors.** `bg-primary`, `text-muted-foreground` — never raw values like `bg-blue-500`.

## Critical Rules (always enforced)

### Styling & Tailwind
- **`className` for layout, not styling.** Never override component colors or typography.
- **No `space-x-*` or `space-y-*`.** Use `flex` with `gap-*`.
- **`size-*` when width and height are equal.** `size-10` not `w-10 h-10`.
- **`truncate` shorthand**, not `overflow-hidden text-ellipsis whitespace-nowrap`.
- **No manual `dark:` color overrides.** Use semantic tokens.
- **`cn()` for conditional classes**, not template-literal ternaries.
- **No manual `z-index` on overlays.** Dialog/Sheet/Popover handle their own stacking.

### Component structure
- **Items inside their Group.** `SelectItem` → `SelectGroup`. `DropdownMenuItem` → `DropdownMenuGroup`.
- **Dialog, Sheet, Drawer always need a Title** (`className="sr-only"` if hidden).
- **Full Card composition.** `CardHeader`/`CardTitle`/`CardContent`/`CardFooter`.
- **Button has no `isPending`/`isLoading`.** Compose `Spinner` + `data-icon` + `disabled`.
- **`Avatar` always needs `AvatarFallback`.**

### Use components, not custom markup
- **Empty states use `Empty`.** Callouts use `Alert`. Loading uses `Skeleton`. Badges use `Badge`.
- **`Separator`** not `<hr>` or `<div className="border-t">`.
- **Forms use `FieldGroup` + `Field`.** Option sets (2–7) use `ToggleGroup`.

### Icons
- **Icons in `Button` use `data-icon`** (`inline-start`/`inline-end`).
- **No sizing classes on icons inside components.**
- **Pass icons as objects**, not string keys.

## Component selection

| Need | Use |
| --- | --- |
| Button/action | `Button` + variant |
| Form inputs | `Input`, `Select`, `Combobox`, `Switch`, `Checkbox`, `RadioGroup`, `Textarea`, `Slider` |
| Toggle 2–5 options | `ToggleGroup` + `ToggleGroupItem` |
| Data display | `Table`, `Card`, `Badge`, `Avatar` |
| Navigation | `Sidebar`, `Tabs`, `Breadcrumb`, `Pagination` |
| Overlays | `Dialog` (modal), `Sheet` (side), `Drawer` (bottom), `AlertDialog` |
| Feedback | `toast`, `Alert`, `Progress`, `Skeleton`, `Spinner` |
| Menus | `DropdownMenu`, `ContextMenu`, `Menubar` |
| Tooltips | `Tooltip`, `HoverCard`, `Popover` |
| Command palette | `Command` inside `Dialog` |

## Workflow

1. `npx shadcn@latest info` for context.
2. Check installed components before `add` (don't re-add).
3. `npx shadcn@latest search` to find components.
4. `npx shadcn@latest docs <component>` for docs/examples, then fetch.
5. `npx shadcn@latest add` to install. Use `--dry-run` / `--diff` when updating.
6. Always read added files and verify correctness (missing sub-components, imports, violations).
7. Never guess the registry — ask when unspecified.
8. Never `--overwrite` without explicit approval.

## Design tokens

See `spec/design-tokens.md` in this repo for the enforced two-level token policy.
