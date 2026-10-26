# Scaffold: component

## Where components go

TODO(team): e.g.
- Shared, reusable across pages: `src/components/ui/`
- Feature-specific: `src/features/<feature>/components/`

Before creating a component, search for an existing one that does the same thing (or almost) -
extending it is usually better than a near-duplicate.

## Checklist

- [ ] File/folder naming matches neighbours - TODO(team): e.g. `PascalCase.tsx`, one component per file, `index.ts` barrel?
- [ ] Props typed with an explicit interface/type; no `any`
- [ ] Styling via TODO(team): Tailwind / CSS modules / styled-components - same as neighbours
- [ ] No data fetching inside presentational components (TODO(team): adjust to your rule)
- [ ] Accessible: buttons are `<button>`, inputs have labels, images have alt
- [ ] Exported from the barrel file if the folder has one
- [ ] Storybook story if the project uses Storybook - TODO(team)
- [ ] Test if it has logic/interaction

## Team notes

TODO(team)
