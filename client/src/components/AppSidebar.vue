<!--
  SKILL TEMPLATE: AppSidebar.vue (not imported by this repo's app).

  A future run of the saas-sidebar-redesign skill copies this into the target
  app's client/src/components/ and adapts it. The adapter must change:
    1. navItems     - routes (`to`), label keys (`labelKey`) and icon names to
                      match the target app's router and i18n catalog.
    2. Brand text   - wire the `title` / `subtitle` props from the app shell,
                      e.g. :title="t('nav.companyName')" :subtitle="t('nav.subtitle')".
    3. Footer slot  - place the language switcher, profile menu, etc. inside
                      <template #footer> where <AppSidebar> is used.
    4. App shell    - offset main content by var(--sidebar-width) (see the
                      note on `.sidebar` below) and import the copied tokens
                      file (src/styles/tokens.css) once from the app entry.
  Uses only the skill's design tokens; no hardcoded colors.
-->
<template>
  <aside class="sidebar" aria-label="Main navigation">
    <div class="sidebar-brand">
      <h1 class="brand-title">{{ title }}</h1>
      <span class="brand-subtitle">{{ subtitle }}</span>
    </div>

    <nav class="sidebar-nav" aria-label="Primary">
      <ul class="nav-list">
        <li v-for="item in navItems" :key="item.to">
          <!--
            exact-active-class drives the highlight and aria-current="page".
            Plain active-class would make '/' match every route (all paths
            start with '/'), so '/' gets an empty active-class and is only
            highlighted on an exact match.
          -->
          <router-link
            :to="item.to"
            class="nav-link"
            :active-class="item.to === '/' ? '' : 'is-active'"
            exact-active-class="is-active"
          >
            <svg
              class="nav-icon"
              viewBox="0 0 20 20"
              width="20"
              height="20"
              fill="none"
              stroke="currentColor"
              stroke-width="1.75"
              stroke-linecap="round"
              stroke-linejoin="round"
              aria-hidden="true"
              focusable="false"
            >
              <path
                v-for="d in iconPaths[item.icon]"
                :key="d"
                :d="d"
              />
            </svg>
            <span class="nav-label">{{ t(item.labelKey) }}</span>
          </router-link>
        </li>
      </ul>
    </nav>

    <!--
      Footer is a slot so the sidebar stays presentational: the app shell owns
      the language switcher and profile menu (and their events/state) and
      simply drops them in here.
    -->
    <div class="sidebar-footer">
      <slot name="footer" />
    </div>
  </aside>
</template>

<script>
export default {
  name: 'AppSidebar',
  props: {
    // Translation function from the app's i18n composable, called as t(key)
    t: {
      type: Function,
      required: true
    },
    title: {
      type: String,
      default: 'Company Name'
    },
    subtitle: {
      type: String,
      default: 'Inventory Management'
    }
  },
  setup() {
    // Static navigation config: `to` is the route path, `labelKey` is the
    // i18n key resolved via the `t` prop, `icon` selects an entry in iconPaths.
    const navItems = [
      { to: '/', labelKey: 'nav.overview', icon: 'overview' },
      { to: '/inventory', labelKey: 'nav.inventory', icon: 'inventory' },
      { to: '/orders', labelKey: 'nav.orders', icon: 'orders' },
      { to: '/spending', labelKey: 'nav.finance', icon: 'finance' },
      { to: '/demand', labelKey: 'nav.demandForecast', icon: 'demand' },
      { to: '/reports', labelKey: 'nav.reports', icon: 'reports' }
    ]

    // Icon name -> list of SVG path strings (20x20 viewBox, stroked).
    // Kept as data so no icon library is needed; add an entry per new icon.
    const iconPaths = {
      // Four-tile grid
      overview: [
        'M3 3h5.5v5.5H3z',
        'M11.5 3H17v5.5h-5.5z',
        'M3 11.5h5.5V17H3z',
        'M11.5 11.5H17V17h-5.5z'
      ],
      // Box / package
      inventory: [
        'M10 2.5l7 3.5v8l-7 3.5-7-3.5V6z',
        'M3 6l7 3.5L17 6',
        'M10 9.5v8'
      ],
      // Clipboard with lines
      orders: [
        'M6.5 4H5a1.5 1.5 0 0 0-1.5 1.5v11A1.5 1.5 0 0 0 5 18h10a1.5 1.5 0 0 0 1.5-1.5v-11A1.5 1.5 0 0 0 15 4h-1.5',
        'M7 2.5h6v3H7z',
        'M7 10h6',
        'M7 13.5h4'
      ],
      // Coin with currency stroke
      finance: [
        'M10 2.5a7.5 7.5 0 1 0 0 15 7.5 7.5 0 0 0 0-15z',
        'M12.25 7.5c-.4-.6-1.2-1-2.25-1-1.4 0-2.25.7-2.25 1.6 0 2.3 4.5 1 4.5 3.3 0 .9-.9 1.6-2.25 1.6-1.05 0-1.85-.4-2.25-1',
        'M10 5v1.5',
        'M10 13.5V15'
      ],
      // Trend line with arrow
      demand: [
        'M2.5 14.5l5-5 3 3 6-6.5',
        'M12.5 6h4v4'
      ],
      // Document with lines
      reports: [
        'M5 2.5h6.5L16 7v9.5a1 1 0 0 1-1 1H5a1 1 0 0 1-1-1v-13a1 1 0 0 1 1-1z',
        'M11.5 2.5V7H16',
        'M7 11h6',
        'M7 14h4'
      ]
    }

    // Everything referenced in the template must be returned
    return {
      navItems,
      iconPaths
    }
  }
}
</script>

<style scoped>
/*
  Fixed to the viewport so it never scrolls with page content. Because fixed
  elements leave normal flow, the app shell MUST offset main content with
  margin-left: var(--sidebar-width) (and drop any top-nav padding), or the
  content will render underneath the sidebar.
*/
.sidebar {
  position: fixed;
  top: 0;
  left: 0;
  bottom: 0;
  width: var(--sidebar-width);
  display: flex;
  flex-direction: column;
  background: var(--sidebar-bg);
  border-right: 1px solid var(--sidebar-border);
  z-index: var(--z-sidebar);
}

.sidebar-brand {
  display: flex;
  flex-direction: column;
  gap: var(--space-1);
  padding: var(--space-5) var(--space-4);
  border-bottom: 1px solid var(--sidebar-border);
  /* Brand stays put; only the nav region scrolls */
  flex-shrink: 0;
}

.brand-title {
  margin: 0;
  font-size: var(--font-size-lg);
  font-weight: 600;
  color: var(--color-ink);
  line-height: 1.25;
}

.brand-subtitle {
  font-size: var(--font-size-xs);
  color: var(--color-muted);
}

.sidebar-nav {
  /* flex: 1 + min-height: 0 lets this region shrink and scroll on short
     viewports instead of pushing the footer off screen */
  flex: 1;
  min-height: 0;
  overflow-y: auto;
  padding: var(--space-3) var(--space-2);
}

.nav-list {
  list-style: none;
  margin: 0;
  padding: 0;
  display: flex;
  flex-direction: column;
  gap: var(--space-1);
}

.nav-link {
  /* relative so the ::before indicator bar anchors to the link */
  position: relative;
  display: flex;
  align-items: center;
  gap: var(--space-3);
  min-height: var(--sidebar-item-height);
  padding: 0 var(--space-3);
  border-radius: var(--radius-md);
  color: var(--color-text);
  font-size: var(--font-size-sm);
  font-weight: 500;
  text-decoration: none;
  transition: background var(--transition-fast), color var(--transition-fast);
}

.nav-link:hover {
  background: var(--color-hover);
  color: var(--color-ink);
}

/* Visible keyboard focus ring; inset so the scrolling nav does not clip it */
.nav-link:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: -2px;
}

.nav-link.is-active,
.nav-link.is-active:hover {
  background: var(--sidebar-item-active-bg);
  color: var(--color-primary);
}

/* 3px left indicator bar, only rendered for the active item */
.nav-link.is-active::before {
  content: '';
  position: absolute;
  left: 0;
  top: var(--space-2);
  bottom: var(--space-2);
  width: 3px;
  border-radius: var(--radius-sm);
  background: var(--sidebar-indicator);
}

.nav-icon {
  /* Fixed 20px icon; flex-shrink stops long labels from squashing it */
  flex-shrink: 0;
  width: 20px;
  height: 20px;
}

.nav-label {
  /* Truncate long translated labels rather than wrapping the row */
  min-width: 0;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.sidebar-footer {
  display: flex;
  flex-direction: column;
  gap: var(--space-2);
  padding: var(--space-3) var(--space-4);
  border-top: 1px solid var(--sidebar-border);
  /* Footer stays pinned at the bottom regardless of nav height */
  flex-shrink: 0;
}

/* Hide the footer's padding/border when the slot is empty */
.sidebar-footer:empty {
  display: none;
}
</style>
