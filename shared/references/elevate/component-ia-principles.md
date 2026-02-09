# Component IA Principles

> Source: tna Component Catalog (51 components, 7 categories), variants/states system

## 1. 5-Category Classification
| Category | Criterion | Examples |
|----------|-----------|---------|
| data-display | Shows data only | StatCard, DataTable, Badge |
| navigation | Screen navigation/exploration | Sidebar, Breadcrumb, TabGroup |
| forms | User input collection | Form, Input, Select, DatePicker |
| layout | Structure/arrangement | PageHeader, Section, Grid |
| feedback | Status/result notification | Toast, Modal, Alert, Skeleton |

## 2. State Completeness
All data components must define 4 states:
| State | Required | Description |
|-------|----------|-------------|
| default | Yes | Normal data display |
| loading | Yes | Data loading (Skeleton) |
| error | Yes | API failure (retry button) |
| empty | Yes | Zero records (guidance message) |

## 3. Variant System
- Visual/functional variations of same component use variant prop
- Variant names are usage-based: 'default' | 'compact' | 'expanded'
- Maximum 4 variants (exceeding → separate component)

## 4. Props Interface Principles
- required props: Only what's essential for component operation
- optional props: Items with defaults (variant, size, className)
- callback props: on{Event} naming (onClick, onChange, onSubmit)
- Separate data props from UI props (data vs appearance)

## 5. Extraction Threshold
- 3+ repetitions: Must extract as component
- 2 repetitions: Record as candidate (re-evaluate in B3)
- 1 occurrence: Keep inline
