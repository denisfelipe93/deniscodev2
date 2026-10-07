<script setup lang="ts">
import { onMounted, onUnmounted, ref } from "vue";

const visible = ref(false);
const menuOpen = ref(false);
const activeSection = ref("");
const theme = ref<"light" | "dark">("light");

const sections = [
  { id: "solucoes", label: "soluções" },
  { id: "projetos", label: "projetos" },
  { id: "processo", label: "processo" },
  { id: "contato", label: "contato" },
];

let heroObserver: IntersectionObserver | null = null;
let sectionObserver: IntersectionObserver | null = null;

function applyTheme() {
  document.documentElement.dataset.siteTheme = theme.value;
}

function toggleTheme() {
  theme.value = theme.value === "light" ? "dark" : "light";

  applyTheme();
  localStorage.setItem("deniscode-theme", theme.value);
}

function closeMenu() {
  menuOpen.value = false;
}

onMounted(() => {
  const savedTheme = localStorage.getItem("deniscode-theme");

  if (savedTheme === "light" || savedTheme === "dark") {
    theme.value = savedTheme;
  }

  applyTheme();

  const hero = document.querySelector("#top");

  if (hero) {
    heroObserver = new IntersectionObserver(
      ([entry]) => {
        visible.value = !entry.isIntersecting;

        if (!visible.value) {
          menuOpen.value = false;
        }
      },
      {
        threshold: 0.05,
      },
    );

    heroObserver.observe(hero);
  }

  sectionObserver = new IntersectionObserver(
    (entries) => {
      const visibleSections = entries
        .filter((entry) => entry.isIntersecting)
        .sort(
          (a, b) =>
            Math.abs(a.boundingClientRect.top) -
            Math.abs(b.boundingClientRect.top),
        );

      if (visibleSections.length > 0) {
        activeSection.value = visibleSections[0].target.id;
      }
    },
    {
      rootMargin: "-35% 0px -55% 0px",
      threshold: 0,
    },
  );

  sections.forEach(({ id }) => {
    const section = document.getElementById(id);

    if (section) {
      sectionObserver?.observe(section);
    }
  });
});

onUnmounted(() => {
  heroObserver?.disconnect();
  sectionObserver?.disconnect();
});
</script>

<template>
  <header
    class="site-nav"
    :class="{
      'site-nav--visible': visible,
      'site-nav--menu-open': menuOpen,
    }"
  >
    <div class="site-nav__inner">
      <a
        href="#top"
        class="site-nav__brand"
        aria-label="Voltar ao início"
        @click="closeMenu"
      >
        <img
          src="/brand/dc/dc-dark.png"
          alt="DenisCode"
        />
      </a>

      <nav class="site-nav__links" aria-label="Navegação principal">
        <a
          v-for="section in sections"
          :key="section.id"
          :href="`#${section.id}`"
          class="site-nav__link"
          :class="{
            'is-active': activeSection === section.id,
          }"
        >
          {{ section.label }}
        </a>
      </nav>

      <button
        class="site-nav__theme"
        type="button"
        :aria-label="
          theme === 'light'
            ? 'Ativar tema escuro'
            : 'Ativar tema claro'
        "
        @click="toggleTheme"
      >
        <svg
          v-if="theme === 'light'"
          viewBox="0 0 24 24"
          aria-hidden="true"
        >
          <path
            d="M21 12.8A8.5 8.5 0 1 1 11.2 3a6.5 6.5 0 0 0 9.8 9.8Z"
          />
        </svg>

        <svg
          v-else
          viewBox="0 0 24 24"
          aria-hidden="true"
        >
          <circle cx="12" cy="12" r="4" />
          <path d="M12 2v2M12 20v2M4.93 4.93l1.41 1.41M17.66 17.66l1.41 1.41M2 12h2M20 12h2M4.93 19.07l1.41-1.41M17.66 6.34l1.41-1.41" />
        </svg>
      </button>

      <button
        class="site-nav__burger"
        type="button"
        :aria-expanded="menuOpen"
        aria-label="Abrir menu"
        @click="menuOpen = !menuOpen"
      >
        <span></span>
        <span></span>
      </button>
    </div>

    <div class="site-nav__mobile">
      <nav aria-label="Navegação mobile">
        <a
          v-for="section in sections"
          :key="section.id"
          :href="`#${section.id}`"
          :class="{
            'is-active': activeSection === section.id,
          }"
          @click="closeMenu"
        >
          {{ section.label }}
        </a>
      </nav>

      <button
        class="site-nav__mobile-theme"
        type="button"
        @click="toggleTheme"
      >
        <span>
          {{ theme === "light" ? "modo escuro" : "modo claro" }}
        </span>

        <span>
          {{ theme === "light" ? "☾" : "☀" }}
        </span>
      </button>
    </div>
  </header>
</template>

<style scoped>
.site-nav {
  --nav-bg: #050506;
  --nav-text: #ffffff;
  --nav-text-muted: rgba(255, 255, 255, 0.68);

  position: fixed;
  top: 0;
  left: 0;

  width: 100%;
  height: 64px;

  z-index: 100;

  background: var(--nav-bg);
  color: var(--nav-text);

  transform: translateY(-100%);
  visibility: hidden;

  transition:
    transform 220ms ease,
    visibility 220ms ease,
    background-color 220ms ease,
    color 220ms ease;
}

:global(html[data-site-theme="dark"]) .site-nav {
  --nav-bg: #ffffff;
  --nav-text: #111111;
  --nav-text-muted: rgba(17, 17, 17, 0.62);
}

.site-nav--visible {
  transform: translateY(0);
  visibility: visible;
}

.site-nav__inner {
  width: min(100% - 4rem, 1520px);
  height: 100%;

  margin-inline: auto;

  display: flex;
  align-items: center;
}

.site-nav__brand {
  display: flex;
  align-items: center;
}

.site-nav__brand img {
  width: 39px;
  height: auto;
}

.site-nav__links {
  display: flex;
  align-items: center;
  gap: 2.25rem;

  margin-left: auto;
}

.site-nav__link {
  color: var(--nav-text-muted);

  font-size: 0.9rem;
  font-weight: 500;

  transition: color 160ms ease;
}

.site-nav__link:hover {
  color: var(--nav-text);
}

.site-nav__link.is-active {
  color: var(--color-brand);
}

.site-nav__link.is-active:hover {
  color: var(--color-brand-hover);
}

.site-nav__theme {
  width: 36px;
  height: 36px;

  margin-left: 2rem;

  display: grid;
  place-items: center;

  padding: 0;

  border: 0;
  background: transparent;
  color: var(--nav-text);

  cursor: pointer;
}

.site-nav__theme svg {
  width: 19px;
  height: 19px;

  fill: none;
  stroke: currentColor;
  stroke-width: 1.6;
  stroke-linecap: round;
  stroke-linejoin: round;
}

.site-nav__burger {
  display: none;
}

.site-nav__mobile {
  display: none;
}

@media (max-width: 760px) {
  .site-nav {
    height: 58px;
  }

  .site-nav__inner {
    width: calc(100% - 3rem);
  }

  .site-nav__brand img {
    width: 35px;
  }

  .site-nav__links,
  .site-nav__theme {
    display: none;
  }

  .site-nav__burger {
    width: 36px;
    height: 36px;

    margin-left: auto;

    display: flex;
    flex-direction: column;
    justify-content: center;
    gap: 6px;

    padding: 7px;

    border: 0;
    background: transparent;

    cursor: pointer;
  }

  .site-nav__burger span {
    width: 100%;
    height: 1px;

    background: var(--nav-text);

    transition:
      transform 180ms ease,
      opacity 180ms ease;
  }

  .site-nav--menu-open .site-nav__burger span:first-child {
    transform: translateY(3.5px) rotate(45deg);
  }

  .site-nav--menu-open .site-nav__burger span:last-child {
    transform: translateY(-3.5px) rotate(-45deg);
  }

  .site-nav__mobile {
    position: fixed;
    inset: 58px 0 0 0;

    display: flex;
    flex-direction: column;

    padding: 3rem 1.5rem 2rem;

    background: var(--nav-bg);

    opacity: 0;
    visibility: hidden;
    transform: translateY(-10px);

    transition:
      opacity 180ms ease,
      visibility 180ms ease,
      transform 180ms ease;
  }

  .site-nav--menu-open .site-nav__mobile {
    opacity: 1;
    visibility: visible;
    transform: translateY(0);
  }

  .site-nav__mobile nav {
    display: flex;
    flex-direction: column;
  }

  .site-nav__mobile nav a {
    padding: 0.8rem 0;

    color: var(--nav-text-muted);

    font-size: clamp(2rem, 9vw, 3.25rem);
    letter-spacing: -0.04em;

    transition: color 160ms ease;
  }

  .site-nav__mobile nav a:hover {
    color: var(--nav-text);
  }

  .site-nav__mobile nav a.is-active {
    color: var(--color-brand);
  }

  .site-nav__mobile-theme {
    margin-top: auto;

    display: flex;
    justify-content: space-between;

    padding: 1.25rem 0 0;

    border: 0;
    border-top: 1px solid currentColor;

    background: transparent;
    color: var(--nav-text);

    font: inherit;
    cursor: pointer;

    opacity: 0.7;
  }
}
</style>