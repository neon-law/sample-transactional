---
name: portal-chrome
description: >
  How a Project portal is allowed to look: it wears `@neon-law-source-code/navigator-ux` as shipped (the Neon Law teal,
  `NavigatorShell` with `NavigatorNavbar` and `NavigatorFooter`, and a way back to the matter show page), and it never
  repaints the palette or hand-rolls the frame. Trigger whenever a task touches a portal's styling, colours, header,
  footer, navigation, or page frame, and before adding any stylesheet to `portal/src/styles/`. Kept identical, byte for
  byte, in every Project repository's `.agents/skills/`; this canonical copy lives in Navigator's own `.agents/skills/`.
---

# The portal wears the library, unmodified

Before naming a table, field, state, or UI concept, follow the **Find the term first** section in `AGENTS.md`.

Navigator serves a Project portal from its own host at `/app/projects/<code>/portal/`, one segment below the matter show
page. The portal must look like the product, so it renders `@neon-law-source-code/navigator-ux` exactly as the library
ships it. Everything below follows from that rule.

## The palette is teal, and there is no brand layer

The library ships one identity: the Neon Law teal, as `--nav-teal-*` behind the `--nav-color-*` semantic aliases.
`www.neonlaw.com` is that library with nothing painted over it, and a portal is too.

**Do not add a brand layer.** Do not create `portal/src/styles/brand.css`, do not redeclare `--nav-color-primary` or any
of its siblings, and do not import a stylesheet between the library's and the app's own. A `:root:root` block that
repaints the ramp is the typical mistake: it wins on specificity and repaints every component at once.

If a colour needs to change, change it in `neon-law-source-code/navigator-ux`, where it is reviewed as a visual
regression across every consumer, never in one portal.

## `app.css` is the seam, and it stays nearly empty

`portal/src/styles/app.css` is imported last and wins on equal specificity. Use it only for the rare thing that must
disagree with the library, such as a grid the library has no component for. It is **not** for tokens: a token restated
there stops tracking the palette and breaks on the next library release.

Comment anything added there with what it disagrees with and why the library could not do it.

## The frame is `NavigatorShell`, not a hand-rolled header

Use the chrome the library ships:

- **`NavigatorShell`** wraps the page, with `header` and `footer` slots.
- **`NavigatorNavbar`** is the header. Give it `brand`, `destinations` for the portal's own routes, and a `brandHref`
  that leaves the portal (the part most often missed).
- **`NavigatorFooter`** is the footer: a host `legal` line, then the fixed
  `Powered by Neon Law Navigator` line (`POWERED_BY_NEON_LAW_NAVIGATOR`), then `links` and `release`. Do not pass the
  platform wording as `legal`; the library renders it.

Do not build a bar out of primitives, and do not leave the portal with no way out. `PageHeader`, `CaseHead`, and `Hero`
are page headings, not chrome: put them inside the shell, never in place of it.

## The way back is the matter show page

`brandHref` points at `/app/projects/<code>`, the matter show page one segment above the mount. Navigator serves it, and
it is the authority on everything a portal declines to state: the status, the participations, the answers, and the
documents.

Derive it from the mount rather than writing it out:

```ts
export function matterShowPath(base: string): string {
  return `/app/projects/${projectCode(base)}`
}
```

`import.meta.env.BASE_URL` is the mount and the only source of an in-app path. A Vite base rewrites module and asset
URLs but never a hand-written href, so a literal path survives the build and can point at another Project's matter.

Every other href stays under the mount, with **no trailing slash**. Navigator's `asset_path_is_safe` requires every
`/`-separated segment to be non-empty, so `notations/` is refused *before* the single-page-application fallback with the
same non-disclosing 404 a nonparticipant receives. A broken link looks exactly like a permission boundary.

## Two component traps

- **`Runs` is inline.** It emits spans and no block of its own, so two adjacent `Runs` render as one run-on line with
  no separator. For two lines of text, use `Prose` with one entry per paragraph.
- **`FactCard` and `DataTable`'s `empty` slot wrap their children in a paragraph.** A block component nested inside is
  invalid markup a browser silently reflows. Put a `Runs` in a `FactCard`, and use `Empty` for a list with nothing in it
  rather than the table's own sentence-sized slot.

## Themes are the library's too

`ThemeProvider` reports the operating system's colour scheme; the tokens do the rest in a media query. Add no toggle, no
stored choice, and no pre-paint script. An SVG drawn into a portal uses `var(--nav-color-…)` for every fill and stroke,
because a hex value will not follow the theme.

## Portal tests never pin wording

A portal test checks structure, never what the copy says. The source of truth for wording is the notation and the data
constants imported from it, and a lawyer reviews that wording in the source diff. A test that restates it adds no safety
and forces a second edit every time a legal term changes.

**Never write:**

- A literal wording assertion, `toMatch` or its `.not` form, on a copy constant or on rendered text.
- A literal string in a query: a role query's `name`, `getByText`, `getByLabelText`, or any other text lookup.
- A pinned case name, citation, or quotation.

**Do write**, because each one checks structure:

- Text the test needs, taken from the imported data value: `name: section.label`, `getByText(CONSTANT)`.
- Ids and counts.
- Section order, compared by slug.
- Every data item renders.
- Routing and mount.
- Links stay on the allowlist, and nothing loads from an outside host.
- No unrendered `{{`.
- A citation is not a URL, and a version has its format.

A security check on the rendered page is not wording. Keep it as it is.

When a test needs a string the data does not supply, assert the structure that produced it instead of copying the string
into the test. Do not paper over a failing wording assertion by rewording it to match: delete it.

## Before you open a pull request

```bash
navigator project gate
cd portal && pnpm check
```

`navigator project gate` reads `project:` from `navigator.yaml`. Without that key the layout half does not run. With it,
a checkout whose directory name is not the Project code still validates against the manifest.
