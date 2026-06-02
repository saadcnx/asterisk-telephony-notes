# 🔬 docs-debugging-lab

> A hands-on debugging lab for real-world documentation issues — intentionally broken examples with step-by-step fixes for Markdown, MDX, broken links, sidebar errors, image paths, and deployment problems.

![Status](https://img.shields.io/badge/status-actively%20maintained-brightgreen)
![Type](https://img.shields.io/badge/type-debugging%20lab-orange)
![Focus](https://img.shields.io/badge/focus-docs%20engineering-blue)

---

## 📌 What This Repo Is

This is not a notes repo. This is a **lab**.

Every folder contains:
- ❌ A **broken** example — something that would actually fail in a real docs project
- ✅ A **fixed** version — with the exact change that resolved it
- 📝 A **write-up** — explaining what went wrong, why, and how to find it

The goal is to build real debugging instinct for documentation systems — the kind of skill that matters in developer support and docs engineering roles.

---

## 🗂️ Repository Structure

```text
docs-debugging-lab/
│
├── broken-mdx-examples/
│   ├── broken/
│   │   └── example.mdx          ← file with intentional MDX error
│   ├── fixed/
│   │   └── example.mdx          ← corrected version
│   └── notes.md                 ← what broke, why, how to fix
│
├── fixed-mdx-examples/
│   └── ...                      ← clean reference MDX patterns
│
├── broken-links/
│   ├── broken/
│   ├── fixed/
│   └── notes.md
│
├── image-path-issues/
│   ├── broken/
│   ├── fixed/
│   └── notes.md
│
├── sidebar-errors/
│   ├── broken/
│   ├── fixed/
│   └── notes.md
│
├── deployment-errors/
│   ├── logs/                    ← example error logs
│   ├── fixes/
│   └── notes.md
│
└── README.md
```

---

## 🧪 Lab Sections

### 💥 broken-mdx-examples
Common MDX mistakes that break page rendering — and how to catch them.

| Error Type | Example |
|---|---|
| Invalid JSX syntax | Unclosed component tags |
| Bad import path | Wrong relative path to a component |
| Missing frontmatter | No `title` or `description` field |
| Unsupported HTML in MDX | Using raw `<div class="">` instead of `className` |
| Export conflict | Multiple default exports in one file |

---

### ✅ fixed-mdx-examples
Clean, working MDX patterns for reference. Use these when you need to know what correct looks like.

---

### 🔗 broken-links
Documenting dead links, incorrect anchor references, and cross-page link failures.

| Error Type | Example |
|---|---|
| Absolute path used instead of relative | `/docs/guide` instead of `../guide` |
| Anchor link pointing to deleted heading | `#old-section-title` |
| Typo in file path | `instalation.md` instead of `installation.md` |
| Link to renamed file | File moved but reference not updated |

---

### 🖼️ image-path-issues
Image not loading? This section covers the most common reasons why.

| Error Type | Example |
|---|---|
| Wrong relative path | `../../images/logo.png` resolving incorrectly |
| File name case mismatch | `Logo.png` vs `logo.png` on Linux servers |
| Missing file extension | `![img](./screenshot)` |
| Image in wrong public folder | Static assets not in `/public` or `/static` |

---

### 📂 sidebar-errors
Sidebar navigation config mistakes that break the docs structure.

| Error Type | Example |
|---|---|
| Missing file reference | File exists but not added to sidebar config |
| Wrong slug | Slug in config doesn't match file path |
| Duplicate ID | Two sidebar items with the same key |
| Invalid YAML/JSON syntax | Missing comma or wrong indentation in config |

---

### 🚀 deployment-errors
Build failures and deployment issues with real error log examples and fixes.

| Error Type | Example |
|---|---|
| Build fails on MDX parse error | Syntax error caught only in production build |
| Environment variable missing | `NEXT_PUBLIC_API_URL` not set in deployment |
| Wrong Node version | `package.json` requires Node 18, server runs 16 |
| Missing dependency | Package in code but not in `package.json` |

---

## 🔍 How Each Fix Is Documented

Every section follows this format:

**Problem:** What the error looks like (error message or broken output)
**Cause:** Why it happens
**Investigation:** How to find it — what to check first
**Fix:** The exact change that resolves it
**Prevention:** How to avoid it next time

---

## 🎯 Goal

Build the debugging instinct to quickly identify why a docs site is broken — and explain the fix clearly to a developer or a support ticket.

---

## 📈 Status

Actively maintained. New broken examples and fixes added regularly as I encounter real-world documentation issues.
