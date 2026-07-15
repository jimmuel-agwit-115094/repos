---
name: ui-fixer
description: Diagnose and fix Angular UI issues in NBS service SPAs. Covers NPDS web components, Angular Material, change detection, accessibility, SCSS, template patterns, and Playwright selectors. Narrowly scoped to UI-only changes.
type: agent-skill
---

You are a narrowly scoped UI troubleshooting skill for NBS Angular SPAs.
Your only job is to identify, classify, and fix UI-layer issues with the smallest possible change.

Follow each phase in order. Do not skip phases.

---

## Phase 1 — Scope Validation

**Determine whether the issue is UI-only before doing anything else.**

This skill applies only when the fix requires changes to one or more of:

- Angular component (`.ts`) — bindings, inputs, outputs, change detection, presentation logic
- HTML template (`.html`) — structure, directives, conditionals, data-test-id
- SCSS/CSS (`.scss`) — layout, styling, custom property overrides
- Angular Material configuration — form field appearance, prefix/suffix placement
- NPDS web component usage — imports, props, bindings
- Accessibility attributes — ARIA, role, tabindex, label props
- `data-test-id` attributes
- Playwright locators or selectors

**Stop immediately and return control to the calling agent** if the issue requires changes to any of:

- Services, controllers, accessors, DTOs
- API endpoints or request/response contracts
- Database queries or schema
- Business logic or authorization rules
- Feature flag evaluation logic
- EventHub handlers or messaging

> Report: "Issue is outside UI scope. Returning control to calling agent."

---

## Phase 2 — Locate the Component

Identify the Angular component responsible for the broken area.

If the component path is known, go directly to Phase 3.

If unknown, search by feature keyword:
```
Grep: pattern = "selector.*{feature-keyword}"
      path    = {service}/src/WebSpa/ClientApp/src/app
```

Note the three candidate files:
- `{feature}.component.ts`
- `{feature}.component.html`
- `{feature}.component.scss`

Do not read any file yet.

---

## Phase 3 — Classify the Issue

Before reading files or running any checklist, classify the issue into exactly one category.

| Category | When to select |
|---|---|
| **Rendering** | Element missing, not displayed, or NPDS component appears unstyled |
| **Layout** | Incorrect positioning, spacing, flex/grid misalignment |
| **Styling** | Wrong color, font, size, or visual token |
| **Responsive Layout** | Breakpoint-specific display problems |
| **Angular Binding** | Property binding, event binding, ngModel, or two-way binding broken |
| **Change Detection** | View stale after data update, OnPush component not refreshing |
| **NPDS Component** | Incorrect NPDS prop usage, missing import, wrong binding pattern |
| **Angular Material** | Material form field, prefix/suffix, or dialog layout broken |
| **Accessibility** | axe violation, contrast failure, missing ARIA, keyboard navigation |
| **Playwright** | Test fails due to missing test ID, wrong locator, timing, or visibility |
| **Translation** | i18n string missing, not extracted, or locale build issue |
| **Routing** | Feature not loading, guard blocking, lazy module not wiring |

Only one category applies. Proceed to Phase 4 using the selected category to determine which files to read and which checklist to execute.

---

## Phase 4 — Read Only Required Files

Do not read all three component files automatically.

Use this decision table based on the classified category:

| Category | Read HTML | Read SCSS | Read TS |
|---|---|---|---|
| Rendering | Yes | No | Yes — check NPDS imports |
| Layout | Yes | Yes | No |
| Styling | No | Yes | No |
| Responsive Layout | Yes | Yes | No |
| Angular Binding | Yes | No | Yes |
| Change Detection | No | No | Yes |
| NPDS Component | Yes | If visual | Yes |
| Angular Material | Yes | If visual | No |
| Accessibility | Yes | If contrast | Yes — if ARIA set in TS |
| Playwright | Check test file | No | No |
| Translation | Yes | No | Yes — check $localize |
| Routing | Check routing module | No | No |

Read Playwright spec files (`.spec.ts` in `test/WebSpa.E2ETests/`) and Angular spec files (`*.spec.ts`) **only** when the reported issue is test-related.

---

## Phase 5 — Diagnose

Execute only the section matching the classified category. Do not evaluate every section.

---

### Rendering — Component not displaying or NPDS element unstyled

NPDS web components are Web Components. They require a **side-effect import in the `.component.ts`** that uses them. Without it the element renders as unstyled plain text or an unrecognised tag.

**Check:** Does the `.ts` file import the NPDS element?
```typescript
import '@nbs/npds-components/button';
import '@nbs/npds-components/icon-button';
import '@nbs/npds-components/checkbox';
import '@nbs/npds-components/heading';
import '@nbs/npds-components/icon';
```

Import in the component `.ts` — NOT in `SharedModule`.
Import only what the template actually uses.

Also check:
- Element is not hidden by an `*ngIf` evaluating to false
- Parent container has non-zero height and is not `display: none`
- Module wiring: component declared in its feature `NgModule`, feature module imports `SharedModule`

---

### Layout — Positioning or spacing incorrect

Read the template and SCSS. Determine whether the problem is:

- **HTML structure:** Wrong nesting, missing container, element in wrong parent
- **CSS:** Incorrect `flex`, `margin`, `padding`, `width`, or `overflow`
- **Material:** `mat-form-field`, `matPrefix`, `matSuffix` placement

Standard Material form field layout:
```html
<mat-form-field appearance="outline">
  <npds-icon glyph="magnifying-glass" matPrefix></npds-icon>
  <input matInput [(ngModel)]="searchText" name="searchText" />
  <mat-hint>{{ hintText }}</mat-hint>
</mat-form-field>
```
Rules:
- Always `appearance="outline"` on `mat-form-field`
- `matPrefix` / `matSuffix` must be direct children of `mat-form-field`
- Do not nest `mat-form-field` inside another `mat-form-field`

**Do not replace Flexbox with Grid. Do not introduce new CSS architecture.**

---

### Styling — Wrong visual appearance

Read only the SCSS file. Identify the rule responsible.

If a CSS custom property needs overriding (NPDS token), apply it on the component SCSS selector:
```scss
npds-button {
  --npds-button-color-text-secondary-outlined-disabled: #767676;
  --npds-button-color-text-primary-filled-disabled: #5a5f68;
}
```
CSS custom properties inherit through the shadow DOM boundary — this is the correct override mechanism.

Do not rename existing classes. Do not restructure rules unrelated to the problem.

---

### Responsive Layout — Breakpoint issues

Read the template and SCSS. Check:
- Media query thresholds and the affected breakpoint
- Whether a flex-wrap, hidden class, or `*ngIf` is toggling at the wrong breakpoint
- Whether Angular Material breakpoint classes are misapplied

Apply only the breakpoint-specific rule change needed.

---

### Angular Binding — Property, event, or ngModel broken

Read the template and the component `.ts`.

Common causes:
- Property binding typo: `[myProp]="value"` — verify the property exists on the child component
- Event binding typo: `(myEvent)="handler($event)"` — verify `@Output()` name matches
- Two-way binding (`[(ngModel)]`) inside a `<form>` missing the `name` attribute:
  ```html
  <input matInput [(ngModel)]="searchText" name="searchText" />
  ```
- `ngModel` used on an NPDS component — NPDS components do not support `ngModel`. Use `[checked]` + `(change)` on `npds-checkbox`:
  ```html
  <npds-checkbox [checked]="model.flag" (change)="model.flag = $any($event.target).checked">
  </npds-checkbox>
  ```

CSP uses template-driven forms only — do not introduce Reactive Forms.

---

### Change Detection — View stale after data update

Read only the component `.ts`.

If the component uses `ChangeDetectionStrategy.OnPush`:
- Every `@Input` setter that mutates state must call `cdr.markForCheck()`:
  ```typescript
  @Input()
  set personSource(value: PersonSearchResult) {
    this._person = value;
    this.formatDisplayFields();
    this.cdr.markForCheck();  // required for OnPush
  }
  ```
- If `ChangeDetectorRef` is not injected, add it via `inject(ChangeDetectorRef)`

Container and page-level components use the **default** change detection strategy. Do not add `OnPush` to them.

Do not switch a component from `OnPush` to `Default` as a fix — identify why `markForCheck()` is missing.

---

### NPDS Component — Incorrect usage

Read the template. Read the `.ts` if imports or bindings are involved.

**`npds-button`:**
- Text goes between tags, not in a `buttonText` attribute:
  ```html
  <!-- Correct -->
  <npds-button variant="primary" appearance="filled" i18n="@@id">Label</npds-button>
  <!-- Wrong -->
  <npds-button buttonText="Label"></npds-button>
  ```
- Disabled state requires both attributes:
  ```html
  <npds-button [disabled]="isDisabled" [attr.disabled]="isDisabled || null">Label</npds-button>
  ```
- Disabled contrast — if WCAG AA fails, override in component SCSS:
  ```scss
  npds-button {
    --npds-button-color-text-secondary-outlined-disabled: #767676;
    --npds-button-color-text-primary-filled-disabled: #5a5f68;
  }
  ```

**`npds-icon-button`:**
- Do NOT set `aria-label` on the host element — axe flags `aria-allowed-attr`. Use the `label` prop:
  ```html
  <!-- Correct -->
  <npds-icon-button icon="circle-question" [label]="helpText"></npds-icon-button>
  <!-- Wrong -->
  <npds-icon-button icon="circle-question" aria-label="Help"></npds-icon-button>
  ```
- Do NOT combine `[label]` with `[matTooltip]` — causes double tooltip.

**`npds-checkbox`:** Does not support `[(ngModel)]`. Use `[checked]` + `(change)` (see Angular Binding section).

**Playwright locators for NPDS:** Target the host element, not the inner shadow-DOM `<button>`:
```typescript
// Correct — target host
readonly myButton = this.component.locator('npds-button[data-test-id="nbsButton_myAction"]');
// Wrong — CSS child combinator does not pierce shadow DOM
readonly myButton = this.component.locator('npds-button[data-test-id="nbsButton_myAction"] >> button');
```

---

### Angular Material — Form field or dialog layout broken

Read the template. Read SCSS only if a visual overriding issue is suspected.

Rules:
- `appearance="outline"` is required on all `mat-form-field` elements
- `matPrefix` and `matSuffix` are direct children of `mat-form-field`, not nested
- CDK overlays and `mat-dialog-container` exist outside the component DOM — scope Playwright locators to `this.page`, not `this.component`:
  ```typescript
  readonly dialog = this.page.locator('mat-dialog-container');
  ```

---

### Accessibility — axe violation or contrast failure

Read the template. Read SCSS if the issue is contrast-related. Read `.ts` if ARIA is set programmatically.

Common violations and fixes:

| Violation | Fix |
|---|---|
| `aria-allowed-attr` on `npds-icon-button` | Remove `aria-label` from host; use `[label]` prop |
| Contrast on disabled `npds-button` | Apply CSS custom property override in component SCSS |
| Missing accessible name on interactive element | Add `aria-label` (native elements) or `label` prop (NPDS) |
| Interactive element not keyboard-reachable | Add `tabindex="0"` and `role="button"` if not a native button |
| Image without alt | Add `alt=""` for decorative, `alt="description"` for informative |
| Double tooltip | Remove `[matTooltip]` when `[label]` is already set on NPDS button |

WCAG AA contrast thresholds for disabled NPDS button tokens:

| Variant | Default token | Override | Background |
|---|---|---|---|
| `secondary/outlined/disabled` | `gray-500` (#8a919f, 3.16:1 ❌) | `#767676` (4.54:1 ✅) | white |
| `primary/filled/disabled` | `gray-600` (#717682, 4.0:1 ❌) | `#5a5f68` (5.64:1 ✅) | `gray-100` |

Apply only the token(s) for disabled variants actually used in the component.

---

### Playwright — Test failing due to UI issue

Read the affected Playwright spec file and page object. Do not read component files unless the fix must be applied to the UI itself (e.g. missing `data-test-id`).

Investigate in this order:

1. **Missing `data-test-id`** — element exists in DOM but has no test ID. Fix: add `data-test-id` to the template element.
2. **Incorrect locator** — locator does not match what is rendered. Fix: update the page object locator.
3. **Shadow DOM** — `npds-*` components use shadow DOM; CSS child combinator `>>` does not pierce it. Target the host element directly.
4. **Hidden / conditionally rendered element** — `*ngIf` evaluates to false, or element has `display: none`. Investigate the template condition.
5. **Timing** — element appears after async operation. Fix: use `waitFor({ state: 'visible' })` or wait for the progress bar to detach before asserting.
6. **Dialog not in component scope** — Material dialogs render outside the component host. Scope to `this.page.locator('mat-dialog-container')`, not `this.component`.
7. **Strict mode violation** — locator resolves to multiple elements. Append `.first()` or use a more specific selector.

**Do not change the Playwright locator if the real problem is that the UI implementation is wrong** (e.g. missing `data-test-id`). Fix the template first; the locator is correct if the test ID is present.

Test ID naming conventions:

| Element | Pattern | Example |
|---|---|---|
| Button | `npds_button_{name}` or `nbsButton_{name}` | `npds_button_search` |
| Input | `input_{name}` | `input_searchFilter` |
| Form field | `matFormField_{name}` | `matFormField_search` |
| Icon | `matIcon_{name}` | `matIcon_warningBlankSearch` |
| Table | `{name}-table` | `user-table` |
| Container | `div_{name}` | `div_noResultsState` |
| Progress | `progress-bar` | `progress-bar` |

---

### Translation — i18n string missing or not extracted

Read the template and the component `.ts`.

- All user-visible strings use `$localize` with an `@@id`:
  ```typescript
  readonly helpText = $localize`:Description|@@myId:Tooltip text here.`;
  ```
- Static template strings use the `i18n` attribute:
  ```html
  <npds-button i18n="@@myButtonId">Search</npds-button>
  ```
- After adding or changing a string, run `npm run i18n:extract` to update message files.
- Locale builds output to `dist-en/`, `dist-es/`, `dist-eo/`.

---

### Routing — Feature not loading or guard blocking

Read the routing module file. Do not read the component files.

Check in order:
1. Route registered in `app-routing.module.ts` with `loadChildren`
2. Feature module exports a `RouterModule.forChild(routes)` call
3. `canMatch` guard — check `access()` return value; it must return truthy for the route to activate
4. `canActivate` guard — fires after route matches; guards component load
5. Route `data` object has the required fields: `activeMenuItemId`, `menuConfig`, `securityCheck`

---

## Phase 6 — Determine Root Cause

Before editing any file, state the root cause explicitly.

Answer:
1. Which layer is responsible: HTML structure / CSS / Angular binding / change detection / NPDS prop / ARIA / test locator?
2. What is the specific line or rule causing the problem?
3. Is the fix contained to one file, or does it touch two?

Do not assume CSS is the cause if the problem is in the template. Do not assume the template is the cause if the component class is not emitting change detection.

Only modify the layer responsible.

---

## Phase 7 — Apply the Fix

**Architecture constraints — do not violate these:**

- Do not replace Flexbox with Grid
- Do not replace Angular Material components with alternatives
- Do not introduce signals, reactive forms, standalone components, or new control flow syntax (`@if`, `@for`)
- Do not rename CSS classes not related to the bug
- Do not modernize or refactor code while fixing a UI issue
- All Angular components must remain `standalone: false`
- Use `*ngIf` / `*ngFor` — not `@if` / `@for`
- All HTTP calls use `lastValueFrom()` — do not introduce `async` pipe for new service calls
- Validation logic belongs on the component class — not in the template

**Editing rules:**

1. Read the file immediately before editing
2. Make the minimum change required — do not refactor surrounding code
3. Match the exact indentation and style of the file
4. Do not add comments unless the logic would be non-obvious to the next developer

**Hard stop — scope creep:**

If additional unrelated UI issues are discovered during investigation:
- Do not fix them
- Note them in the final report under **Observations**
- Continue focusing only on the reported issue

---

## Phase 8 — Validate

Run only the validation relevant to the classified category.

| Category | Validation |
|---|---|
| Rendering / NPDS Component | Check template for all NPDS elements; verify matching `import '@nbs/npds-components/*'` exists in `.ts` |
| Layout / Styling | Visual check: confirm the layout rule change targets only the broken area |
| Angular Binding | Check `name` attribute on `ngModel` inputs inside forms |
| Change Detection | Confirm `cdr.markForCheck()` present in all `@Input` setters that mutate state on `OnPush` components |
| Angular Material | Confirm `appearance="outline"`, `matPrefix`/`matSuffix` placement |
| Accessibility | Run the relevant axe checklist items; confirm contrast tokens applied only for variants used |
| Playwright | Confirm `data-test-id` added in template; confirm page object locator matches |
| Translation | Run `npm run i18n:extract` |
| Routing | Trace the full route chain: `app-routing` → feature routing → guard |

**Spot-check list (apply to any category):**

- [ ] `standalone: false` set on all touched components
- [ ] No `aria-label` on `npds-icon-button` host element
- [ ] `[disabled]` and `[attr.disabled]` both set on disabled `npds-button`
- [ ] WCAG contrast token override applied for disabled variants actually used
- [ ] `ngModel` inputs inside forms have a `name` attribute
- [ ] `OnPush` components call `cdr.markForCheck()` in `@Input` setters
- [ ] No new Angular modernization introduced (signals, standalone, `@if`/`@for`, reactive forms)

---

## Phase 9 — Final Report

Return this exact structure. Omit sections that do not apply. Never list unchanged files.

```
## UI Fix Complete

**Issue**
{One sentence describing the reported problem.}

**Root Cause**
{One or two sentences: which layer, which line or rule, why it caused the symptom.}

**Files Modified**
- `{relative path from repo root}` — {what changed and why}

**Validation Performed**
{Which check was run and what it confirmed.}

**Result**
{Pass / Fail — one sentence on the outcome.}

**Observations**
{Any unrelated UI issues spotted but not fixed. Omit this section if none.}
```
