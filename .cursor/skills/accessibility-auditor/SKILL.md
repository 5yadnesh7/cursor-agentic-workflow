---
name: accessibility-auditor
description: Audits web and app UI for accessibility against WCAG 2.2 AA, covering keyboard operability, focus management, semantics and ARIA, color contrast, motion/animation safety, forms and error handling, media alternatives, and assistive-tech compatibility. Produces severity-tagged findings with concrete remediations. Use when the user asks for an a11y audit, before launching user-facing UI, or when reviewer / dev-engineer / tester flags accessibility concerns.
---

# Accessibility Auditor

Specialist a11y review for web and app UIs. Goes beyond automated scans: validates real keyboard, screen-reader, and low-vision experiences against WCAG 2.2 AA and common platform guidelines.

## When To Use

- Pre-launch or pre-release audits of user-facing UI.
- After significant UI changes affecting navigation, forms, or media.
- Complaints or reports of inaccessible experiences.
- `reviewer` or `tester` escalates a11y concerns.

## Inputs

- `target`: URL, route, or component set.
- `scope`: full site, specific flow, or component library.
- `standard`: default WCAG 2.2 AA; adjust if the org requires AAA or platform-specific rules.
- `context_path` (default `.cursor/context/PROJECT_CONTEXT.md`).

## Prerequisite: Project Context

Run the Context Gate; invoke `get-project-context` to confirm the frontend stack, design system, i18n setup, and component boundaries.

## Core Principles

1. Accessibility is a product requirement, not a QA afterthought.
2. Automated scans catch ~30% of issues; manual testing finds the rest.
3. Test with real keyboard and screen-reader flows, not just attributes.
4. Use platform semantics first; ARIA only when semantics do not suffice.
5. Respect user preferences (reduced motion, high contrast, zoom).
6. Accessible defaults scale better than per-page fixes.

## Workflow

```
A11y Audit:
- [ ] Step 1: Ensure project context
- [ ] Step 2: Run automated scan as a starting signal
- [ ] Step 3: Keyboard pass
- [ ] Step 4: Screen-reader pass
- [ ] Step 5: Visual and motion pass
- [ ] Step 6: Forms and error handling pass
- [ ] Step 7: Media and content pass
- [ ] Step 8: Structure and landmarks pass
- [ ] Step 9: Mobile and touch pass
- [ ] Step 10: Report
```

### Step 1: Ensure Project Context

Know the framework, routing, design system tokens, i18n strategy, and any existing a11y conventions.

### Step 2: Automated Scan

Use available automated tools (axe-core, Lighthouse, Pa11y, WAVE, platform inspectors) as a quick triage. Treat results as candidates, not conclusions.

### Step 3: Keyboard Pass

- Every interactive element is reachable via Tab in a logical order.
- No keyboard traps.
- Visible focus indicator on every focusable element; contrast sufficient.
- Skip links to main content and major regions.
- Custom widgets implement the correct key interactions (menus, tabs, dialogs, comboboxes, trees, grids) per APG patterns.
- `Esc` closes dialogs, menus, and popovers; focus returns to the trigger.

### Step 4: Screen-Reader Pass

Test with at least one screen reader per target platform (NVDA + JAWS on Windows, VoiceOver on macOS/iOS, TalkBack on Android).

- Page has a meaningful, unique title.
- Headings form a logical hierarchy.
- Landmarks (`header`, `nav`, `main`, `aside`, `footer`) are used.
- Every control has an accessible name; avoid generic names ("button", "link").
- Live regions announce updates without being noisy.
- Dynamic content changes are announced or focus is managed appropriately.
- Icons that convey meaning have text alternatives; decorative icons are hidden.

### Step 5: Visual And Motion Pass

- Text contrast ≥ 4.5:1 (3:1 for large text) against background.
- Non-text UI components and graphical objects ≥ 3:1.
- Text remains readable at 200% zoom without horizontal scrolling.
- Content reflows at 320px width without loss.
- Respect `prefers-reduced-motion` and `prefers-color-scheme`.
- No content flashes more than three times per second.
- Color is never the only means of conveying information.

### Step 6: Forms And Error Handling

- Every input has a programmatic label, not just placeholder text.
- Required fields are indicated both visually and programmatically.
- Error messages are associated with their inputs and announced.
- Validation is understandable and recoverable; do not strip user input.
- Autocomplete tokens where helpful; avoid blocking paste on password fields.
- Groupings via `fieldset`/`legend` for radio/checkbox sets.

### Step 7: Media And Content

- Images: meaningful `alt` text; decorative images marked as such.
- Icons with meaning: text alternative via accessible name.
- Video: captions for dialogue, transcripts when feasible, audio descriptions for significant visuals.
- Audio: transcripts.
- Auto-playing media avoided or easy to pause.
- Language attribute set on `html`; per-passage language if multilingual.

### Step 8: Structure And Landmarks

- Single `main` landmark.
- Consistent navigation across pages.
- Consistent identification of components with the same function.
- Page regions named when multiple of the same landmark appear.
- Heading outline reflects document structure.

### Step 9: Mobile And Touch

- Touch targets ≥ 44×44 CSS px (platform equivalent) with adequate spacing.
- Gestures have single-pointer alternatives.
- Content works in both portrait and landscape without loss.
- Dynamic type / text size changes are respected.
- Screen-reader gestures work with custom components.

### Step 10: Report

```
# Accessibility Audit: <scope>

## Standard
WCAG 2.2 AA (+ <platform guidelines if applicable>)

## Context Used
- PROJECT_CONTEXT.md (last updated: <ts>)

## Summary
- 🔴 Critical: <n>
- 🟠 High: <n>
- 🟡 Medium: <n>
- 🟢 Low: <n>

## Findings
### 🔴 Critical (blocks a user from completing the task)
- **<page/component>** — <issue> — **WCAG:** <SC> — **Fix:** <concrete>

### 🟠 High
- ...

### 🟡 Medium
- ...

### 🟢 Low / Informational
- ...

## Positive Notes
- <what is already accessible>

## Systemic Recommendations
- <design-system, tokens, lint rules, component primitives>

## Verification Plan
- <manual re-test with keyboard + screen reader>
- <automated checks in CI>
```

## Severity Guidance

| Severity | Meaning |
|----------|---------|
| 🔴 Critical | A user with a disability cannot complete a core task. |
| 🟠 High | Significant barrier or frequent friction; fix before release. |
| 🟡 Medium | Noticeable issue with workaround; fix in planned window. |
| 🟢 Low | Minor polish, consistency, or best-practice improvement. |

## Coordination

- Component-level fixes → `dev-engineer`, usually in the design system.
- Content changes (alt text, labels, transcripts) → content owners via `doc-writer` where appropriate.
- Automated checks in CI → `devops` to wire into pipelines.
- Manual regression tests → `tester` (ask before writing automated flows).
- Major redesigns → `brainstorm`.

## Quality Bar

- Findings cite the specific WCAG success criterion or platform guideline.
- Every finding has a reproducible step and a concrete remediation.
- Recommendations prefer fixing the design system over fixing pages one by one.
- Automated + manual coverage is named and sustained (CI + periodic audits).
- No finding is "make it more accessible" without a specific ask.

## Anti-Patterns

- Relying only on automated scanners.
- Sprinkling ARIA to "patch" broken semantics instead of using correct elements.
- `aria-label`s that duplicate visible text inconsistently.
- Removing focus outlines without replacing them.
- Placeholder-as-label.
- Animations that ignore reduced-motion.
- Treating accessibility as a launch-blocking checklist rather than a design property.
