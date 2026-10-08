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

function toggleMenu() {
  menuOpen.value = !menuOpen.value;

  document.documentElement.style.overflow = menuOpen.value
    ? "hidden"
    : "";
}

function closeMenu() {
  menuOpen.value = false;
  document.documentElement.style.overflow = "";
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
          closeMenu();
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

  document.documentElement.style.overflow = "";
});
</script>

<template>
  <header
    class="site-nav"
    :class="{
      'site-nav--visible': visible,
      'site-nav--menu-open': menuOpen,
      'site-nav--dark': theme === 'dark',
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

      <nav
        class="site-nav__links"
        aria-label="Navegação principal"
      >
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

          <path
            d="M12 2v2M12 20v2M4.93 4.93l1.41 1.41M17.66 17.66l1.41 1.41M2 12h2M20 12h2M4.93 19.07l1.41-1.41M17.66 6.34l1.41-1.41"
          />
        </svg>
      </button>

      <button
        class="site-nav__burger"
        type="button"
        :aria-expanded="menuOpen"
        :aria-label="
          menuOpen
            ? 'Fechar menu'
            : 'Abrir menu'
        "
        @click="toggleMenu"
      >
        <span></span>
        <span></span>
      </button>
    </div>

    <div class="site-nav__mobile">
      <nav
        class="site-nav__mobile-links"
        aria-label="Navegação mobile"
      >
        <a
          v-for="(section, index) in sections"
          :key="section.id"
          :href="`#${section.id}`"
          class="site-nav__mobile-link"
          :class="{
            'is-active': activeSection === section.id,
          }"
          @click="closeMenu"
        >
          <span class="site-nav__mobile-index">
            {{ String(index + 1).padStart(2, "0") }}
          </span>

          <span class="site-nav__mobile-label">
            {{ section.label }}
          </span>

          <span
            class="site-nav__mobile-arrow"
            aria-hidden="true"
          >
            ↗
          </span>
        </a>
      </nav>

      <div class="site-nav__mobile-footer">
        <button
          class="site-nav__mobile-theme"
          type="button"
          @click="toggleTheme"
        >
          <span>
            {{
              theme === "light"
                ? "modo escuro"
                : "modo claro"
            }}
          </span>

          <span aria-hidden="true">
            {{ theme === "light" ? "☾" : "☀" }}
          </span>
        </button>

        <p class="site-nav__mobile-signature">
          deniscode<span>.</span>

          <small>
            tecnologia aplicada a problemas reais.
          </small>
        </p>
      </div>
    </div>
  </header>
</template>

<style scoped>
.site-nav {
  --nav-bg: rgba(245, 245, 243, 0.88);
  --nav-bg-solid: #f4f4f1;

  --nav-text: #111111;
  --nav-text-muted: rgba(17, 17, 17, 0.56);

  --nav-line: rgba(17, 17, 17, 0.075);
  --nav-hover: rgba(17, 17, 17, 0.045);

  position: fixed;
  top: 0;
  left: 0;

  width: 100%;
  height: 68px;

  z-index: 1000;

  background: var(--nav-bg);
  color: var(--nav-text);

  border-bottom: 1px solid var(--nav-line);

  backdrop-filter: blur(18px);
  -webkit-backdrop-filter: blur(18px);

  transform: translateY(-100%);
  visibility: hidden;

  transition:
    transform 240ms ease,
    visibility 240ms ease,
    background-color 220ms ease,
    border-color 220ms ease,
    color 220ms ease;
}

.site-nav--dark {
  --nav-bg: rgba(9, 9, 11, 0.94);
  --nav-bg-solid: #0b0b0e;

  --nav-text: #f5f5f5;
  --nav-text-muted: rgba(255, 255, 255, 0.58);

  --nav-line: rgba(255, 255, 255, 0.075);
  --nav-hover: rgba(255, 255, 255, 0.045);
}

.site-nav--visible {
  transform: translateY(0);
  visibility: visible;
}

/* ========================================
   INNER
   ======================================== */

.site-nav__inner {
  position: relative;
  z-index: 2;

  width: min(
    calc(100% - 6rem),
    var(--container-width)
  );

  height: 100%;

  margin-inline: auto;

  display: flex;
  align-items: center;
}

/* ========================================
   BRAND
   ======================================== */

.site-nav__brand {
  display: flex;
  align-items: center;

  opacity: 0.96;

  transition:
    opacity 160ms ease,
    transform 180ms ease;
}

.site-nav__brand:hover {
  opacity: 1;
  transform: translateY(-1px);
}

.site-nav__brand img {
  width: 38px;
  height: auto;

  filter: invert(1);
}

.site-nav--dark .site-nav__brand img {
  filter: none;
}

/* ========================================
   DESKTOP LINKS
   ======================================== */

.site-nav__links {
  display: flex;
  align-items: center;

  gap: 2.4rem;

  margin-left: auto;
}

.site-nav__link {
  position: relative;

  display: flex;
  align-items: center;

  height: 68px;

  color: var(--nav-text-muted);

  font-size: 0.82rem;
  font-weight: 500;
  line-height: 1;

  letter-spacing: -0.01em;

  text-decoration: none;

  transition: color 180ms ease;
}

.site-nav__link::after {
  content: "";

  position: absolute;

  left: 0;
  bottom: 0;

  width: 100%;
  height: 1px;

  background: var(--nav-text);

  opacity: 0.55;

  transform: scaleX(0);
  transform-origin: left;

  transition:
    transform 180ms ease,
    background-color 180ms ease,
    opacity 180ms ease;
}

.site-nav__link:hover {
  color: var(--nav-text);
}

.site-nav__link:hover::after {
  transform: scaleX(1);
}

.site-nav__link.is-active {
  color: var(--nav-text);
}

.site-nav__link.is-active::after {
  background: var(--color-brand);

  opacity: 1;

  transform: scaleX(1);
}

/* ========================================
   THEME
   ======================================== */

.site-nav__theme {
  width: 34px;
  height: 34px;

  margin-left: 1.8rem;

  display: grid;
  place-items: center;

  padding: 0;

  border: 1px solid transparent;
  border-radius: 50%;

  background: transparent;
  color: var(--nav-text-muted);

  cursor: pointer;

  transition:
    color 180ms ease,
    background-color 180ms ease,
    border-color 180ms ease,
    transform 180ms ease;
}

.site-nav__theme:hover {
  color: var(--nav-text);

  background: var(--nav-hover);
  border-color: var(--nav-line);

  transform: rotate(8deg);
}

.site-nav__theme svg {
  width: 17px;
  height: 17px;

  fill: none;
  stroke: currentColor;

  stroke-width: 1.6;
  stroke-linecap: round;
  stroke-linejoin: round;
}

/* ========================================
   BURGER / MOBILE DEFAULT
   ======================================== */

.site-nav__burger {
  display: none;
}

.site-nav__mobile {
  display: none;
}

/* ========================================
   MOBILE
   ======================================== */

@media (max-width: 760px) {
  .site-nav {
    height: 62px;
  }

  /*
    Quando o menu estiver aberto, o header
    também fica sólido para virar uma única
    superfície com o painel abaixo.
  */
  .site-nav--menu-open {
    background: var(--nav-bg-solid);

    backdrop-filter: none;
    -webkit-backdrop-filter: none;
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

  /* BURGER */

  .site-nav__burger {
    width: 38px;
    height: 38px;

    margin-left: auto;

    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;

    gap: 6px;

    padding: 8px;

    border: 0;
    border-radius: 50%;

    background: transparent;

    cursor: pointer;

    transition: background-color 180ms ease;
  }

  .site-nav__burger:hover {
    background: var(--nav-hover);
  }

  .site-nav__burger span {
    display: block;

    width: 20px;
    height: 1px;

    background: var(--nav-text);

    transform-origin: center;

    transition:
      transform 200ms ease,
      opacity 180ms ease;
  }

  .site-nav--menu-open
    .site-nav__burger
    span:first-child {
    transform:
      translateY(3.5px)
      rotate(45deg);
  }

  .site-nav--menu-open
    .site-nav__burger
    span:last-child {
    transform:
      translateY(-3.5px)
      rotate(-45deg);
  }

  /* ======================================
     MENU PANEL
     ====================================== */

  .site-nav__mobile {
    position: absolute;

    top: 100%;
    left: 0;

    z-index: 1;

    display: flex;
    flex-direction: column;

    width: 100%;
    height: calc(100dvh - 62px);

    padding:
      clamp(2rem, 5vh, 3rem)
      1.5rem
      1.5rem;

    background: var(--nav-bg-solid);

    overflow-y: auto;
    overscroll-behavior: contain;

    opacity: 0;
    visibility: hidden;
    pointer-events: none;

    transform: translateY(-8px);

    transition:
      opacity 180ms ease,
      visibility 180ms ease,
      transform 200ms ease;
  }

  .site-nav--menu-open
    .site-nav__mobile {
    opacity: 1;
    visibility: visible;
    pointer-events: auto;

    transform: translateY(0);
  }

  /* ======================================
     LINKS
     ====================================== */

  .site-nav__mobile-links {
    display: flex;
    flex-direction: column;

    border-top:
      1px solid
      var(--nav-line);
  }

  .site-nav__mobile-link {
    display: grid;

    grid-template-columns:
      2.8rem
      minmax(0, 1fr)
      auto;

    align-items: center;

    min-height: 76px;

    padding: 0;

    border-bottom:
      1px solid
      var(--nav-line);

    color: var(--nav-text);

    text-decoration: none;

    transition:
      color 180ms ease,
      padding-left 180ms ease,
      background-color 180ms ease;
  }

  .site-nav__mobile-index {
    color: var(--nav-text-muted);

    font-size: 0.66rem;
    font-weight: 500;

    line-height: 1;
    letter-spacing: 0.08em;

    transition: color 180ms ease;
  }

  .site-nav__mobile-label {
    font-size:
      clamp(
        1.7rem,
        7vw,
        2.2rem
      );

    font-weight: 400;
    line-height: 1;

    letter-spacing: -0.045em;
  }

  .site-nav__mobile-arrow {
    color: var(--nav-text-muted);

    font-size: 0.9rem;
    line-height: 1;

    opacity: 0;

    transform:
      translate(-5px, 5px);

    transition:
      opacity 180ms ease,
      transform 180ms ease,
      color 180ms ease;
  }

  .site-nav__mobile-link:hover,
  .site-nav__mobile-link.is-active {
    padding-left: 0.3rem;
  }

  .site-nav__mobile-link:hover
    .site-nav__mobile-arrow,
  .site-nav__mobile-link.is-active
    .site-nav__mobile-arrow {
    opacity: 1;

    transform: translate(0, 0);
  }

  .site-nav__mobile-link.is-active
    .site-nav__mobile-index {
    color: var(--color-brand);
  }

  .site-nav__mobile-link.is-active
    .site-nav__mobile-arrow {
    color: var(--color-brand);
  }

  /* ======================================
     FOOTER
     ====================================== */

  .site-nav__mobile-footer {
    margin-top: auto;

    padding-top: 2.5rem;
  }

  .site-nav__mobile-theme {
    width: 100%;

    display: flex;
    align-items: center;
    justify-content: space-between;

    padding: 1rem 0;

    border: 0;
    border-top:
      1px solid
      var(--nav-line);

    border-bottom:
      1px solid
      var(--nav-line);

    background: transparent;
    color: var(--nav-text-muted);

    font: inherit;

    font-size: 0.75rem;
    line-height: 1;

    letter-spacing: 0.02em;

    cursor: pointer;

    transition: color 180ms ease;
  }

  .site-nav__mobile-theme:hover {
    color: var(--nav-text);
  }

  .site-nav__mobile-theme
    span:last-child {
    color: var(--nav-text);

    font-size: 1rem;
  }

  .site-nav__mobile-signature {
    margin: 1.35rem 0 0;

    color: var(--nav-text);

    font-size: 0.72rem;
    font-weight: 500;
    line-height: 1;

    letter-spacing: -0.015em;
  }

  .site-nav__mobile-signature > span {
    color: var(--color-brand);
  }

  .site-nav__mobile-signature small {
    display: block;

    max-width: 210px;

    margin-top: 0.4rem;

    color: var(--nav-text-muted);

    font-size: 0.65rem;
    font-weight: 400;
    line-height: 1.45;

    letter-spacing: 0;
  }
}
</style>