<script setup lang="ts">
import { onMounted, onUnmounted, ref } from "vue";

interface Service {
  id: string;
  title: string;
  description: string;
  intro: string;
  capabilities: string[];
}

defineProps<{
  services: Service[];
}>();

const activeService = ref<Service | null>(null);
const open = ref(false);

function openDrawer(event: Event) {
  const customEvent = event as CustomEvent<Service>;

  activeService.value = customEvent.detail;
  open.value = true;

  document.documentElement.classList.add("service-drawer-open");
  document.body.classList.add("service-drawer-open");
}

function closeDrawer() {
  open.value = false;

  document.documentElement.classList.remove("service-drawer-open");
  document.body.classList.remove("service-drawer-open");

  window.setTimeout(() => {
    activeService.value = null;
  }, 320);
}

function handleKeydown(event: KeyboardEvent) {
  if (event.key === "Escape" && open.value) {
    closeDrawer();
  }
}

onMounted(() => {
  window.addEventListener("deniscode:open-service", openDrawer);
  window.addEventListener("keydown", handleKeydown);
});

onUnmounted(() => {
  window.removeEventListener("deniscode:open-service", openDrawer);
  window.removeEventListener("keydown", handleKeydown);

  document.documentElement.classList.remove("service-drawer-open");
  document.body.classList.remove("service-drawer-open");
});
</script>

<template>
  <div
    class="service-drawer"
    :class="{ 'service-drawer--open': open }"
    :aria-hidden="!open"
  >
    <button
      class="service-drawer__backdrop"
      type="button"
      aria-label="Fechar detalhes"
      @click="closeDrawer"
    ></button>

    <aside
      class="service-drawer__panel"
      role="dialog"
      aria-modal="true"
      :aria-label="activeService?.title"
    >

      <template v-if="activeService">
        <header class="service-drawer__heading">
          <h2>{{ activeService.title }}</h2>

          <p class="service-drawer__intro">
            {{ activeService.intro }}
          </p>
        </header>

        <div class="service-drawer__divider"></div>

        <ul class="service-drawer__list">
          <li
            v-for="(capability, index) in activeService.capabilities"
            :key="capability"
          >
            <span class="service-drawer__item-index">
              {{ String(index + 1).padStart(2, "0") }}
            </span>

            <span class="service-drawer__item-label">
              {{ capability }}
            </span>
          </li>
        </ul>

        <div class="service-drawer__footer">
            <a
                class="service-drawer__cta"
                href="#contato"
                @click="closeDrawer"
            >
                <span>vamos conversar</span>
                <span class="service-drawer__cta-arrow" aria-hidden="true">↗</span>
            </a>

            <button
                class="service-drawer__close"
                type="button"
                aria-label="Fechar"
                @click.stop="closeDrawer"
            >
                <span></span>
                <span></span>
            </button>
            </div>
      </template>
    </aside>
  </div>
</template>

<style scoped>
:global(html.service-drawer-open),
:global(body.service-drawer-open) {
  overflow: hidden;
}

/* =========================
   ROOT
   ========================= */

.service-drawer {
  position: fixed;
  inset: 0;

  z-index: 9999;

  isolation: isolate;

  visibility: hidden;
  pointer-events: none;
}

.service-drawer--open {
  visibility: visible;
  pointer-events: auto;
}

/* =========================
   BACKDROP
   ========================= */

.service-drawer__backdrop {
  position: absolute;
  inset: 0;

  z-index: 0;

  width: 100%;
  height: 100%;

  margin: 0;
  padding: 0;
  border: 0;

  background: rgba(5, 5, 6, 0.64);

  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);

  opacity: 0;

  cursor: default;

  transition: opacity 280ms ease;
}

.service-drawer--open .service-drawer__backdrop {
  opacity: 1;
}

/* =========================
   PANEL
   ========================= */

.service-drawer__panel {
  position: absolute;
  display: flex;
  flex-direction: column;
  top: 0;
  right: 0;

  z-index: 1;

  width: min(820px, 58vw);
  height: 100%;

  padding:
    clamp(6rem, 8vw, 8rem)
    clamp(3rem, 5vw, 5.5rem)
    4rem;

  overflow-y: auto;
  overscroll-behavior: contain;

  background: #f5f5f2;
  color: #111111;

  transform: translateX(100%);

  box-shadow:
    -40px 0 80px rgba(0, 0, 0, 0.12);

  transition:
    transform 340ms cubic-bezier(0.22, 1, 0.36, 1),
    background-color 220ms ease,
    color 220ms ease;
}

:global(html[data-site-theme="dark"]) .service-drawer__panel {
  background: #09090a;
  color: #f4f4f4;

  box-shadow:
    -40px 0 90px rgba(0, 0, 0, 0.45);
}

.service-drawer--open .service-drawer__panel {
  transform: translateX(0);
}

/* linha estrutural mínima */

.service-drawer__panel::before {
  content: "";

  position: absolute;
  top: 0;
  left: 0;

  width: 1px;
  height: 100%;

  background: var(--color-brand);

  opacity: 0.45;
}

.service-drawer__footer {
  margin-top: 2.75rem;

  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 2rem;
  min-height: 44px;
}

/* =========================
   CLOSE
   ========================= */

.service-drawer__close {
  position: relative;
  flex: 0 0 auto;

  width: 44px;
  height: 44px;

  padding: 0;

  border: 1px solid
    color-mix(in srgb, currentColor 12%, transparent);

  background: transparent;
  color: currentColor;

  cursor: pointer;

  transition:
    border-color 160ms ease,
    background-color 160ms ease;
}

.service-drawer__close:hover {
  border-color: color-mix(
    in srgb,
    currentColor 35%,
    transparent
  );

  background:
    color-mix(in srgb, currentColor 4%, transparent);
}

.service-drawer__close span {
  position: absolute;
  top: 50%;
  left: 50%;

  width: 18px;
  height: 1px;

  background: currentColor;
}

.service-drawer__close span:first-child {
  transform: translate(-50%, -50%) rotate(45deg);
}

.service-drawer__close span:last-child {
  transform: translate(-50%, -50%) rotate(-45deg);
}

/* =========================
   HEADING
   ========================= */

.service-drawer__heading {
  max-width: 520px;
}

.service-drawer h2 {
  margin: 0;

  font-size: clamp(3rem, 4.5vw, 5.4rem);
  line-height: 0.94;

  font-weight: 400;
  letter-spacing: -0.055em;
}

.service-drawer__intro {
  max-width: 470px;

  margin: 2rem 0 0;

  color:
    color-mix(
      in srgb,
      currentColor 58%,
      transparent
    );

  font-size: 1.05rem;
  line-height: 1.65;
}

/* =========================
   DIVIDER
   ========================= */

.service-drawer__divider {
  width: 100%;
  height: 1px;

  margin: 3.5rem 0 1.5rem;

  background:
    color-mix(
      in srgb,
      currentColor 10%,
      transparent
    );
}

/* =========================
   CAPABILITIES
   ========================= */

.service-drawer__list {
  margin: 0;
  padding: 0;

  list-style: none;
}

.service-drawer__list li {
  display: grid;
  grid-template-columns: 48px 1fr;
  align-items: center;

  min-height: 58px;

  border-bottom: 1px solid
    color-mix(
      in srgb,
      currentColor 8%,
      transparent
    );
}

.service-drawer__item-index {
  color:
    color-mix(
      in srgb,
      currentColor 35%,
      transparent
    );

  font-size: 0.72rem;
  letter-spacing: 0.08em;
}

.service-drawer__item-label {
  font-size: 1rem;
}

/* =========================
   CTA
   ========================= */

.service-drawer__cta {
  display: inline-flex;
  align-items: center;
  gap: 0.75rem;

  min-height: 44px;
  margin-top: 0;

  color: currentColor;

  font-size: 1.02rem;
  line-height: 1;

  transition:
    gap 160ms ease,
    opacity 160ms ease;
}

.service-drawer__cta:hover {
  gap: 1rem;
  opacity: 0.65;
}

.service-drawer__cta-arrow {
  color: var(--color-brand);
}

/* =========================
   MOBILE
   ========================= */

@media (max-width: 760px) {
  .service-drawer__backdrop {
    backdrop-filter: none;
    -webkit-backdrop-filter: none;

    background: rgba(5, 5, 6, 0.7);
  }

  .service-drawer__panel {
    width: 100vw;
    max-width: none;

    padding:
      5.5rem
      1.5rem
      2.5rem;

    background: #f5f5f2;

    box-shadow: none;
  }

  :global(html[data-site-theme="dark"]) .service-drawer__panel {
    background: #09090a;
  }

  .service-drawer__close {
    width: 40px;
    height: 40px;
  }

  .service-drawer__footer {
    margin-top: 2.5rem;
    padding-top: 0;

  }


  .service-drawer h2 {
    font-size: clamp(2.7rem, 12vw, 4rem);
  }

  .service-drawer__intro {
    margin-top: 1.5rem;

    font-size: 0.98rem;
  }

  .service-drawer__divider {
    margin-top: 2.5rem;
  }

  .service-drawer__list li {
    grid-template-columns: 40px 1fr;

    min-height: 56px;
  }

  .service-drawer__cta {
    margin-top: 0;
  }
}
</style>