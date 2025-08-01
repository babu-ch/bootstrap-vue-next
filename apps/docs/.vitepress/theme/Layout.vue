<!-- eslint-disable vue/multi-word-component-names -->
<template>
  <BNavbar v-b-color-mode="'dark'" variant="primary" sticky="top" toggleable="lg" :container="true">
    <BToastOrchestrator />
    <BModalOrchestrator />
    <BPopoverOrchestrator />
    <div class="d-flex gap-2 align-items-center">
      <BNavbarToggle v-b-toggle.sidebar-menu />
      <BNavbarBrand :to="withBase('/')" class="p-0 me-0 me-lg-2">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          viewBox="0 0 953 953"
          width="32"
          height="32"
          version="1.1"
          shape-rendering="geometricPrecision"
          fill-rule="evenodd"
          clip-rule="evenodd"
          focusable="false"
          role="img"
          class="d-block"
        >
          <title>BootstrapVueNext</title>
          <path
            fill="currentColor"
            d="M92 0h769c50 0 92 42 92 92v769c0 50-42 92-92 92H92c-50 0-92-42-92-92V92C0 42 42 0 92 0zm216 710c100 0 160-50 160-133 0-62-44-107-108-113v-3c48-8 86-52 86-102 0-71-55-117-140-117H111v468h197zM195 307h90c50 0 78 23 78 64 0 44-33 68-91 68h-77V307zm0 338V499h90c64 0 98 25 98 73s-33 73-94 73h-94zm503 65l163-468h-90L652 621h-2L531 242h-92l163 468h96z"
          />
        </svg>
      </BNavbarBrand>
    </div>
    <BCollapse is-nav>
      <BNavbarNav>
        <BNav>
          <BNavItem
            v-for="link in headerLinks"
            :key="link.route"
            :to="link.route"
            :active="route.path === `${link.route}.html`"
            class="py-2"
            active-class="active fw-bold"
            >{{ link.label }}</BNavItem
          >
        </BNav>
      </BNavbarNav>
    </BCollapse>
    <div class="d-flex align-items-center gap-2">
      <VPNavBarSearch :class="{dark: colorMode === 'dark'}" />
      <div class="d-flex gap-2 flex-wrap socials">
        <BNav class="d-flex">
          <BNavItem
            v-for="link in headerExternalLinks"
            :key="link.url"
            :href="link.url"
            :link-attrs="{'aria-label': link.label}"
            target="_blank"
            rel="noopener"
            link-classes="py-1 px-0"
          >
            <component :is="link.icon()" height="1.1em" aria-hidden />
          </BNavItem>
          <div class="border border-secondary ms-2 me-3" />
          <ClientOnly>
            <BNavItemDropdown toggle-class="px-0">
              <!-- TODO there is no way to adjust these options, say if you wanted to remove the padding -->
              <template #button-content>
                <component
                  :is="currentIcon"
                  height="1.1em"
                  :aria-label="`Toggle theme (${colorMode})`"
                  class="d-inline-block"
                />
              </template>
              <BDropdownItem
                v-for="el in options"
                :key="el"
                :active="colorMode === el"
                @click="set(el)"
              >
                <component :is="map[el]" /> {{ el }}
              </BDropdownItem>
            </BNavItemDropdown>
          </ClientOnly>
        </BNav>
      </div>
    </div>
  </BNavbar>
  <ClientOnly>
    <div v-if="!isLargeScreen" class="py-4 px-3 text-end">
      <BNavbarToggle v-b-toggle.otp-menu class="otp-menu-toggle">
        On this page
        <ChevronRight aria-hidden />
      </BNavbarToggle>
    </div>
  </ClientOnly>
  <BContainer fluid class="container-lg mt-3 my-md-4 bd-layout">
    <aside class="bd-sidebar">
      <ClientOnly>
        <BOffcanvas
          id="sidebar-menu"
          v-model="sidebar"
          responsive="lg"
          title="Browse docs"
          header-class="d-flex offcanvas-hidden-width"
          class="h-100 border-0"
        >
          <TableOfContentsNav />
        </BOffcanvas>
      </ClientOnly>
    </aside>
    <main class="bd-main">
      <BRow v-if="page.isNotFound">
        <BCol>
          <BContainer class="text-center my-auto p-5">
            <BRow>
              <BCol>
                <h1>Oh No!</h1>
              </BCol>
            </BRow>
            <BRow>
              <BCol>
                <h2>File Not Found</h2>
              </BCol>
            </BRow>
          </BContainer>
        </BCol>
      </BRow>
      <BRow v-else>
        <div class="bd-content">
          <aside ref="_target" class="otp-sidebar">
            <ClientOnly>
              <BOffcanvas
                id="otp-menu"
                v-model="onThisPage"
                responsive="lg"
                placement="end"
                title="On this page"
                class="h-100 border-0"
                header-class="pb-0 d-flex offcanvas-hidden-width"
                body-class="py-2"
              >
                <PageContents :contents="contents" :active-id="activeId" />
              </BOffcanvas>
            </ClientOnly>
          </aside>
          <Content ref="_content" style="height: 100%" class="doc-content" />
        </div>
      </BRow>
    </main>
  </BContainer>
</template>

<script setup lang="ts">
import {
  BCol,
  BCollapse,
  BContainer,
  BDropdownItem,
  BModalOrchestrator,
  BNav,
  BNavbar,
  BNavbarBrand,
  BNavbarNav,
  BNavbarToggle,
  BNavItem,
  BNavItemDropdown,
  BOffcanvas,
  BPopoverOrchestrator,
  BRow,
  BToastOrchestrator,
  useColorMode,
  useScrollspy,
  vBColorMode,
  vBToggle,
} from 'bootstrap-vue-next'
import {
  type ComponentPublicInstance,
  computed,
  inject,
  onMounted,
  ref,
  useTemplateRef,
  watch,
} from 'vue'
import GithubIcon from '~icons/bi/github'
import OpencollectiveIcon from '~icons/logos/opencollective'
import DiscordIcon from '~icons/bi/discord'
import MoonStarsFill from '~icons/bi/moon-stars-fill'
import SunFill from '~icons/bi/sun-fill'
import ChevronRight from '~icons/bi/chevron-right'
import CircleHalf from '~icons/bi/circle-half'
import {useData, useRoute, withBase} from 'vitepress'
import {VPNavBarSearch} from 'vitepress/theme'
import {appInfoKey} from './keys'
import {useMediaQuery} from '@vueuse/core'
import PageContents from '../../src/components/PageContents.vue'
import {type ContentsItem, type HeaderItem} from '../../src/types'

// https://vitepress.dev/reference/runtime-api#usedata
const {page} = useData()
const route = useRoute()

const content = useTemplateRef<ComponentPublicInstance<HTMLElement>>('_content')
const target = useTemplateRef<ComponentPublicInstance<HTMLElement>>('_target')

const {current: activeId, list: items} = useScrollspy(content, target, {
  contentQuery: ':scope h1, :scope > div > [id], #component-reference, .component-reference h3',
  targetQuery: ':scope [href]',
  rootMargin: '0px 0px -25%',
  manual: true,
})

const globalData = inject(appInfoKey, {
  discordUrl: '',
  githubUrl: '',
  githubPackageDirectory: '',
  githubComponentsDirectory: '',
  githubComposablesDirectory: '',
  githubMainBranch: '',
  githubDirectivesDirectory: '',
  opencollectiveUrl: '',
  githubDocsDirectory: '',
})

const isLargeScreen = useMediaQuery('(min-width: 992px)')
const sidebar = ref(false)
const onThisPage = ref(false)

const headerLinks = [
  {
    route: withBase('/docs'),
    label: 'Getting Started',
  },
  {
    route: withBase('/docs/icons'),
    label: 'Icons',
  },
  {
    route: withBase('/docs/types'),
    label: 'Types',
  },
  {
    route: withBase('/docs/reference'),
    label: 'Reference',
  },
  {
    route: withBase('/docs/migration-guide'),
    label: 'Migrate',
  },
]

const headerExternalLinks = [
  {
    url: globalData.githubUrl,
    label: 'Open Our Github',
    icon: () => GithubIcon,
  },
  {
    url: globalData.opencollectiveUrl,
    label: 'Open Our Opencollective',
    icon: () => OpencollectiveIcon,
  },
  {
    url: globalData.discordUrl,
    label: 'Open Our Discord Server',
    icon: () => DiscordIcon,
  },
]

const colorMode = useColorMode({
  persist: true,
})

onMounted(() => {
  watch(
    colorMode,
    (newValue) => {
      if (newValue === 'dark') {
        document.documentElement.classList.add('dark')
      } else {
        document.documentElement.classList.remove('dark')
      }
    },
    {immediate: true}
  )
})

const map = {
  dark: MoonStarsFill,
  light: SunFill,
  auto: CircleHalf,
}

const options = Object.keys(map) as (keyof typeof map)[]

const currentIcon = computed(() => map[colorMode.value])

const set = (newValue: keyof typeof map) => {
  colorMode.value = newValue
}

const headers = computed(() =>
  items.value.map((item) => {
    const rawTag = item.el?.tagName?.toUpperCase() ?? ''
    const isHeading = /^H[1-6]$/.test(rawTag)
    const tag = isHeading ? rawTag : 'DIV'
    const level = tag.startsWith('H') ? parseInt(tag.replace('H', '')) : 3
    return {
      ...item,
      tag,
      level,
    } as HeaderItem
  })
)

const contents = computed(() => {
  const root: ContentsItem[] = []
  const stack: ContentsItem[] = []

  headers.value.forEach((header) => {
    const item = {...header, children: [] as ContentsItem[]} as ContentsItem

    while (stack.length && stack[stack.length - 1].level >= item.level) {
      stack.pop()
    }

    if (stack.length === 0) {
      root.push(item)
    } else {
      stack[stack.length - 1].children.push(item)
    }

    stack.push(item)
  })

  if (root.length !== 1) {
    // Something isn't right if we have no root items or more than one root item
    // eslint-disable-next-line no-console
    console.warn('Unexpected header structure:', headers, 'Root items:', root)
  }

  return root.length > 0 ? root[0] : undefined
})

watch(
  () => route.path,
  () => {
    if (isLargeScreen.value === false) {
      sidebar.value = false
    }
  }
)
</script>

<style lang="scss">
@use 'bootstrap/dist/css/bootstrap.css';

:root {
  --vp-c-brand-1: hsla(237, 31%, 35%, 1);
}

#app {
  --black: #000000;
  --white: #ffffff;
  --pink: #e83e8c;
  --bvn-bg-primary: linear-gradient(
    45deg,
    var(--vp-c-brand-1) 0%,
    hsla(230, 25%, 18%, 1) 72%,
    hsla(220, 19%, 13%, 1) 100%
  );

  svg {
    display: inline;
  }

  .bg-primary {
    background: var(--bvn-bg-primary) !important;
    border-color: var(--bvn-bg-primary);
    color: var(--white);
  }

  @mixin hover-focus-active() {
    &:hover,
    &:focus,
    &:active {
      @content;
    }
  }

  .doc-content > div > {
    h2,
    h3,
    h4,
    h5 {
      margin-bottom: 1rem;
    }

    h1 {
      margin-bottom: 1.5rem;
    }

    h2 {
      margin-top: 3rem;
    }

    h3 {
      margin-top: 2rem;
    }

    h4 {
      margin-top: 1.5rem;
    }

    hr {
      margin: 3rem 0;
    }

    ol {
      list-style-type: decimal;
    }

    ul {
      list-style-type: disc;
    }

    // alternative solution until https://github.com/vuejs/vitepress/pull/4082 is released
    table {
      @extend .table;
      @extend .table-striped;
    }
  }

  .alert {
    p:last-child {
      margin-bottom: 0;
    }

    &:last-child {
      margin-bottom: 0;
    }
  }

  .lead {
    margin-bottom: 1rem;
  }

  .breadcrumb {
    margin-bottom: 0;
  }

  [id^='input-group-'] {
    margin-bottom: 1rem;
  }

  .html {
    border-top: var(--bs-border-width) solid rgba(255, 255, 255, 0.2);
    padding: 1rem 1.25rem;
  }

  .tabs {
    margin: 1rem 0 3rem;

    .card {
      border-top-left-radius: 0;
      border-top-right-radius: 0;
      border-top: 0;

      .card-body {
        margin-bottom: 0;
      }
    }
  }

  .component-reference {
    .display-6 {
      font-size: 1.5rem;
      display: block;
      margin: 0.75rem 0 1rem;
    }

    h2 {
      margin-top: 2rem;
    }

    ul {
      li {
        margin-bottom: 0.4rem;
      }
    }

    h5 > a {
      color: var(--bs-body-color);
      text-decoration: none;
    }

    .table {
      margin-top: 0.5rem;
      font-size: 0.9rem;

      & > tbody > tr > td,
      & > tbody > tr > th,
      & > tfoot > tr > td,
      & > tfoot > tr > th,
      & > thead > tr > td,
      & > thead > tr > th {
        padding: 0.5rem;
      }

      & > tbody > tr > td:first-child {
        color: var(--pink);
      }
    }
  }

  .anchorjs-link {
    text-decoration: none;

    &::after {
      content: '#';
    }
  }

  .card {
    margin-bottom: 3rem;

    pre {
      margin-bottom: 0;
    }

    .card-header {
      h3 {
        margin-top: 0;
      }
    }

    .card-body {
      h4:first-child {
        margin-top: 0;
      }

      .card {
        margin-bottom: 0;
      }

      .list-group {
        max-width: 25rem;
      }
    }

    .icon-lg {
      font-size: 2.65rem;
    }
  }

  .card-link {
    text-decoration: none;
  }

  // Navbar.
  > .navbar {
    color: var(--white);
    box-shadow:
      0 0.5rem 1rem rgba(0, 0, 0, 0.15),
      inset 0 -1px 0 rgba(255, 255, 255, 0.15);

    .nav-link,
    .navbar-brand,
    .btn {
      color: var(--white);
    }

    .socials {
      .nav-link {
        padding: 0 0.5rem;
      }
    }
  }

  .otp-menu-toggle {
    border: none;
    font-size: small;
  }

  [class^='language-'] {
    position: relative;

    .copy {
      position: absolute;
      top: 0;
      right: 0;
      z-index: 3;
      display: block;
      border: none;
      padding: 0;
      outline: inherit;
      justify-content: center;
      align-items: center;
      border-radius: 4px;
      background-color: var(--bs-secondary);
      cursor: pointer;
      background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' height='20' width='20' stroke='rgba(255,255,255)' stroke-width='2' viewBox='0 0 24 24'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' d='M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2M9 5a2 2 0 0 0 2 2h2a2 2 0 0 0 2-2M9 5a2 2 0 0 1 2-2h2a2 2 0 0 1 2 2'/%3E%3C/svg%3E");
      background-position: 50%;
      background-size: 20px;
      background-repeat: no-repeat;
      transition: opacity 0.4s;
      opacity: 0;
      height: 2.5rem;
      width: 2.5rem;

      &::after {
        position: absolute;
        top: 0;
        left: -4.6rem;
        content: 'Copied!';
        padding: 0.6rem;
        border-radius: 4px;
        background-color: var(--bs-secondary);
        opacity: 0;
        color: var(--bs-light);
        font-size: 0.9rem;
      }

      &.copied {
        background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' height='20' width='20' stroke='rgba(255,255,255)' stroke-width='2' viewBox='0 0 24 24'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' d='M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2M9 5a2 2 0 0 0 2 2h2a2 2 0 0 0 2-2M9 5a2 2 0 0 1 2-2h2a2 2 0 0 1 2 2m-6 9 2 2 4-4'/%3E%3C/svg%3E");

        &::after {
          opacity: 1;
        }
      }
    }

    .lang {
      position: absolute;
      top: 4px;
      right: 10px;
      z-index: 2;
      font-size: 12px;
      font-weight: 500;
      transition: color 0.5s;
    }

    &:hover {
      .copy {
        :focus {
          opacity: 1;
        }

        opacity: 1;
      }
    }
  }

  // Sidebar.
  .offcanvas {
    box-shadow:
      0 0.5rem 1rem rgba(0, 0, 0, 0.15),
      inset 0 -1px 0 rgba(255, 255, 255, 0.15);
    .list-group {
      padding: 0 0 1.5rem 0;
      font-size: 0.875em;

      .bd-links-heading {
        svg {
          position: relative;
          top: -1px;
          margin-right: 0.3rem;
        }

        display: block;
        margin: 0 0 0.5rem;
        text-transform: uppercase;

        a {
          text-decoration: none;
        }
      }

      .list-group-item {
        border: none;
        padding: 0;

        a {
          color: var(--bs-body-color);
          display: block;
          padding: 0.13rem 0.7rem 0.13rem 0;
          margin: 0.125rem 0;
          text-decoration: none;
        }
      }
    }
  }
}

// Sidebar onscreen.
@media (min-width: 992px) {
  .bd-layout {
    display: grid !important;
    grid-template-areas: 'sidebar main';
    grid-template-columns: 1fr 5fr;
    gap: 1.5rem;

    .bd-sidebar,
    .otp-sidebar {
      grid-area: sidebar;
      position: -webkit-sticky;
      position: sticky;
      top: 5rem;
      display: block !important;
      height: calc(100vh - 6rem);
      padding-left: 0.25rem;
      margin-left: -0.25rem;
      overflow-y: auto;

      .offcanvas {
        transition: none !important;
        transform: 0;
        position: relative !important;

        .offcanvas-header {
          .btn-close {
            display: none;
          }
        }
      }
    }

    .bd-main {
      .bd-content {
        display: grid;
        grid-area: main;
        grid-template-areas: 'intro toc' 'content toc';
        grid-template-rows: auto 1fr;
        grid-template-columns: 4fr 1fr;
        gap: inherit;

        .doc-content {
          grid-area: content;
          min-width: 1px;
        }

        .otp-sidebar {
          margin-left: 1.25rem;
          grid-area: toc;
        }
      }
    }
  }
}

// Sidebar width.
.bd-sidebar,
.otp-sidebar {
  @media (min-width: 992px) {
    min-width: 12.5rem;
  }

  .offcanvas.offcanvas-start,
  .offcanvas-lg.offcanvas-start,
  .offcanvas.offcanvas-end,
  .offcanvas-lg.offcanvas-end {
    @media (min-width: 992px) {
      width: 12.5rem !important;
    }

    @media (max-width: 991px) {
      .bd-links-nav {
        -moz-column-count: 2;
        column-count: 2;
        -moz-column-gap: 1.5rem;
        column-gap: 1.5rem;
      }
    }
  }

  .offcanvas.offcanvas-end {
    @media (max-width: 991px) {
      max-width: 15rem;
    }
  }
}

// Search
.DocSearch-Button .DocSearch-Search-Icon {
  @media (max-width: 767px) {
    color: var(--white);
  }
}
.VPLocalSearchBox {
  z-index: 1021 !important;
}
.search-keyboard-shortcuts kbd {
  color: inherit;
}

// Code Block
[data-bs-theme='dark'] {
  .vp-code-light {
    display: none;
  }
}
[data-bs-theme='light'] {
  .vp-code-dark {
    display: none;
  }
}

.custom-block {
  margin-bottom: 1rem;
}

@media (min-width: 992px) {
  .offcanvas-header.offcanvas-hidden-width .btn-close {
    display: none !important;
  }
}
</style>
