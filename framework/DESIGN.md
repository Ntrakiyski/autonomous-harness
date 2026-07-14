# DESIGN.md — Visual Identity Token Spec

> **Project-specific.** Fill in for each project. This is a Google DESIGN.md
> token file — YAML front matter (machine-readable) + Markdown body (human
> rationale). Agents read this to produce consistent UI.

---
version: alpha
name: [FILL: Project Name]
description: [FILL: One-sentence brand description]
colors:
  primary: "[FILL: #HEX]"
  secondary: "[FILL: #HEX]"
  accent: "[FILL: #HEX]"
  neutral: "[FILL: #HEX]"
  background: "[FILL: #HEX]"
  text: "[FILL: #HEX]"
  error: "[FILL: #HEX]"
  success: "[FILL: #HEX]"
typography:
  h1:
    fontFamily: "[FILL: Inter, system-ui]"
    fontSize: "[FILL: 2.5rem]"
    fontWeight: 700
    lineHeight: 1.2
  h2:
    fontFamily: "[FILL]"
    fontSize: "[FILL: 2rem]"
    fontWeight: 600
    lineHeight: 1.3
  h3:
    fontFamily: "[FILL]"
    fontSize: "[FILL: 1.5rem]"
    fontWeight: 600
    lineHeight: 1.4
  body:
    fontFamily: "[FILL]"
    fontSize: "[FILL: 1rem]"
    fontWeight: 400
    lineHeight: 1.6
  body-sm:
    fontFamily: "[FILL]"
    fontSize: "[FILL: 0.875rem]"
    fontWeight: 400
    lineHeight: 1.5
  caption:
    fontFamily: "[FILL]"
    fontSize: "[FILL: 0.75rem]"
    fontWeight: 500
    lineHeight: 1.4
    letterSpacing: "0.05em"
spacing:
  xs: "[FILL: 4px]"
  sm: "[FILL: 8px]"
  md: "[FILL: 16px]"
  lg: "[FILL: 24px]"
  xl: "[FILL: 32px]"
  "2xl": "[FILL: 48px]"
rounded:
  none: "0px"
  sm: "[FILL: 4px]"
  md: "[FILL: 8px]"
  lg: "[FILL: 12px]"
  full: "9999px"
components:
  button-primary:
    backgroundColor: "{colors.accent}"
    textColor: "#FFFFFF"
    rounded: "{rounded.md}"
    padding: "{spacing.sm} {spacing.lg}"
  button-secondary:
    backgroundColor: "transparent"
    textColor: "{colors.primary}"
    rounded: "{rounded.md}"
    padding: "{spacing.sm} {spacing.lg}"
  input:
    backgroundColor: "#FFFFFF"
    textColor: "{colors.text}"
    rounded: "{rounded.md}"
    padding: "{spacing.sm} {spacing.md}"
  card:
    backgroundColor: "#FFFFFF"
    rounded: "{rounded.lg}"
    padding: "{spacing.lg}"
---

## Overview

[FILL: Brand tone — 2-3 sentences]

## Colors

- **Primary (`{colors.primary}`):** [FILL: what it's used for]
- **Accent (`{colors.accent}`):** [FILL: the sole driver for interaction]
- **Neutral (`{colors.neutral}`):** [FILL: backgrounds, separators]
- **Error (`{colors.error}`):** [FILL: destructive actions, validation]
- **Success (`{colors.success}`):** [FILL: confirmations, completions]

## Typography

[FILL: Font choices and rationale]

## Layout & Spacing

[FILL: Grid system, responsive breakpoints, spacing conventions]

## Components

[FILL: Add a description for each component defined above.
Variants (hover, active, disabled) go as separate component entries
with related key names, e.g. `button-primary-hover`.]

## Do's and Don'ts

- ✅ Do: [FILL]
- ✅ Do: [FILL]
- ❌ Don't: [FILL]
- ❌ Don't: [FILL]
