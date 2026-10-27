# App shell: sidebar layout

Target structure for the root shell component (usually `src/App.vue`). Every app's shell differs, so treat this as a pattern to adapt, not a file to copy.

## Markup

```vue
<template>
  <div class="app-shell">
    <AppSidebar :t="t" :title="t('nav.companyName')" :subtitle="t('nav.subtitle')">
      <template #footer>
        <!-- Relocate the existing components here; do not rewrite them. -->
        <LanguageSwitcher />
        <ProfileMenu @show-profile-details="..." @show-tasks="..." />
      </template>
    </AppSidebar>

    <div class="app-main">
      <!-- Filter bars and page-level toolbars stay here, above the router view. -->
      <FilterBar />
      <main class="main-content">
        <router-view />
      </main>
    </div>

    <!-- Modals stay at the shell root so they overlay the sidebar too. -->
  </div>
</template>
```

## Global CSS (replaces `.app`, `.top-nav`, `.nav-container`, `.nav-tabs`, `.logo`, `.subtitle`)

```css
/* The sidebar is position: fixed (out of flow), so the main column is offset
   by its width instead of sharing a grid track with it. */
.app-shell {
  min-height: 100vh;
  background: var(--color-bg);
}

.app-main {
  margin-left: var(--sidebar-width);
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.main-content {
  flex: 1;
  width: 100%;
  max-width: 1600px;
  margin: 0 auto;
  padding: var(--space-6) var(--space-7);
}
```

Delete the old top-nav rules entirely, including the `.nav-tabs a.active::after` underline, which is positioned against the old bar.

## Dependent-element checklist

Work through each item; skipping them is the usual source of visual bugs after this redesign.

1. **Sticky elements.** Run `grep -rn "position: *\(sticky\|fixed\)" src`. Anything sticky with `top: <top-nav height>` (commonly 70px) must now use `top: 0`, because the top nav is gone. Give sticky elements `z-index: var(--z-sticky)`.
2. **Modals.** Every `position: fixed` overlay must have `z-index: var(--z-modal)` (or any value above `--z-sidebar`). Verify by opening each modal and confirming the sidebar is dimmed behind it.
3. **Full-width bars.** A filter bar or banner that previously spanned the viewport now lives inside `.app-main`, so it spans only the content column. That is intended; remove any `100vw` widths that would overflow into the sidebar.
4. **Hardcoded heights.** Search for `calc(100vh - 70px)` and similar; the 70px no longer exists.
5. **Routes and labels.** Nav items come from the router and the i18n keys. If a label is hardcoded (for example "Reports"), add an i18n key in every locale file rather than hardcoding it in the sidebar.
6. **Narrow viewports.** The sidebar is fixed-width and not collapsible by design. Note in the final summary that very narrow screens are not handled, rather than silently leaving it broken.
