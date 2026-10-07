<script setup lang="ts">
import { onBeforeUnmount, onMounted, ref } from "vue";

const isOpen = ref(false);
const senderEmail = ref("");

const message = ref(
  "Olá,\n\nGostaria de conversar sobre uma ideia, projeto ou problema e entender como podemos seguir."
);

const recipient = "contato@deniscode.com";

const openDrawer = () => {
  isOpen.value = true;

  document.documentElement.classList.add("email-drawer-open");
  document.body.classList.add("email-drawer-open");
};

const closeDrawer = () => {
  isOpen.value = false;

  document.documentElement.classList.remove("email-drawer-open");
  document.body.classList.remove("email-drawer-open");
};

const handleEscape = (event: KeyboardEvent) => {
  if (event.key === "Escape" && isOpen.value) {
    closeDrawer();
  }
};

const handleOpenEvent = () => {
  openDrawer();
};

const handleSubmit = () => {
  /*
    A interface fica pronta agora.

    Depois vamos conectar isso a um endpoint real
    para enviar sem abrir Gmail / Outlook.
  */

  console.log({
    from: senderEmail.value,
    to: recipient,
    message: message.value,
  });
};

onMounted(() => {
  window.addEventListener("deniscode:open-email", handleOpenEvent);
  window.addEventListener("keydown", handleEscape);
});

onBeforeUnmount(() => {
  window.removeEventListener("deniscode:open-email", handleOpenEvent);
  window.removeEventListener("keydown", handleEscape);

  document.documentElement.classList.remove("email-drawer-open");
  document.body.classList.remove("email-drawer-open");
});
</script>

<template>
  <div
    class="email-drawer"
    :class="{ 'email-drawer--open': isOpen }"
    :aria-hidden="!isOpen"
  >
    <button
      class="email-drawer__backdrop"
      type="button"
      aria-label="Fechar mensagem"
      @click="closeDrawer"
    />

    <aside
      class="email-drawer__panel"
      role="dialog"
      aria-modal="true"
      aria-label="Enviar uma mensagem"
    >
      <header class="email-drawer__header">
        <span>e-mail</span>

        <h2>
          uma mensagem<span>.</span>
        </h2>

        <p>
          deixei o começo pronto. você pode ajustar antes de enviar.
        </p>
      </header>

      <form class="email-drawer__form" @submit.prevent="handleSubmit">
        <div class="email-field">
          <label for="email-from">seu e-mail</label>

          <input
            id="email-from"
            v-model="senderEmail"
            type="email"
            autocomplete="email"
            placeholder="voce@email.com"
            required
          />
        </div>

        <div class="email-field">
          <label for="email-to">para</label>

          <input
            id="email-to"
            :value="recipient"
            type="email"
            tabindex="-1"
            readonly
          />
        </div>

        <div class="email-field email-field--message">
          <label for="email-message">mensagem</label>

          <textarea
            id="email-message"
            v-model="message"
            rows="6"
            required
          />
        </div>

        <div class="email-drawer__actions">
          <button
            class="email-drawer__cancel"
            type="button"
            @click="closeDrawer"
          >
            fechar
          </button>

          <button class="email-drawer__submit" type="submit">
            <span>enviar</span>
            <span aria-hidden="true">↗</span>
          </button>
        </div>
      </form>
    </aside>
  </div>
</template>

<style scoped>
:global(html.email-drawer-open),
:global(body.email-drawer-open) {
  overflow: hidden;
}

.email-drawer {
  position: fixed;
  inset: 0;
  z-index: 10000;

  visibility: hidden;
  pointer-events: none;

  isolation: isolate;
}

.email-drawer--open {
  visibility: visible;
  pointer-events: auto;
}

.email-drawer__backdrop {
  position: absolute;
  inset: 0;
  z-index: 0;

  width: 100%;
  height: 100%;

  margin: 0;
  padding: 0;

  border: 0;

  background: rgba(5, 5, 6, 0.62);

  backdrop-filter: blur(7px);
  -webkit-backdrop-filter: blur(7px);

  opacity: 0;

  transition: opacity 280ms ease;
}

.email-drawer--open .email-drawer__backdrop {
  opacity: 1;
}

.email-drawer__panel {
  --drawer-bg: #f5f5f3;
  --drawer-text: #111111;
  --drawer-muted: rgba(17, 17, 17, 0.55);
  --drawer-line: rgba(17, 17, 17, 0.11);

  position: absolute;
  top: 0;
  right: 0;
  z-index: 1;

  width: min(660px, 48vw);
  height: 100dvh;

  display: flex;
  flex-direction: column;

  padding: clamp(3rem, 4.5vw, 4.5rem);

  overflow-y: auto;

  background: var(--drawer-bg);
  color: var(--drawer-text);

  transform: translateX(100%);

  box-shadow: -40px 0 90px rgba(0, 0, 0, 0.14);

  transition:
    transform 340ms cubic-bezier(0.22, 1, 0.36, 1),
    background-color 220ms ease,
    color 220ms ease;
}

:global(html[data-site-theme="dark"]) .email-drawer__panel {
  --drawer-bg: #09090a;
  --drawer-text: #f5f5f5;
  --drawer-muted: rgba(255, 255, 255, 0.54);
  --drawer-line: rgba(255, 255, 255, 0.1);

  box-shadow: -40px 0 90px rgba(0, 0, 0, 0.38);
}

.email-drawer--open .email-drawer__panel {
  transform: translateX(0);
}

.email-drawer__panel::before {
  content: "";

  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;

  width: 1px;

  background: color-mix(
    in srgb,
    var(--color-brand) 55%,
    transparent
  );
}

.email-drawer__header > span {
  display: block;

  margin-bottom: 2rem;

  font-size: 0.72rem;
  font-weight: 600;
  letter-spacing: 0.28em;
  text-transform: uppercase;

  color: var(--drawer-muted);
}

.email-drawer__header h2 {
  margin: 0;

  font-size: clamp(2.8rem, 4vw, 4.6rem);
  font-weight: 400;
  line-height: 0.92;
  letter-spacing: -0.055em;
}

.email-drawer__header h2 span {
  color: var(--color-brand);
}

.email-drawer__header p {
  max-width: 410px;

  margin: 1.8rem 0 0;

  font-size: 1rem;
  line-height: 1.6;

  color: var(--drawer-muted);
}

/* FORM */

.email-drawer__form {
  display: flex;
  flex-direction: column;

  gap: 1.5rem;

  margin-top: clamp(2.5rem, 4vw, 3.5rem);
}

.email-field {
  display: grid;

  gap: 0.8rem;

  padding-bottom: 1.4rem;

  border-bottom: 1px solid var(--drawer-line);
}

.email-field label {
  font-size: 0.7rem;
  font-weight: 600;
  letter-spacing: 0.22em;
  text-transform: uppercase;

  color: var(--drawer-muted);
}

.email-field input,
.email-field textarea {
  width: 100%;

  padding: 0;

  border: 0;
  outline: none;

  background: transparent;
  color: var(--drawer-text);

  font: inherit;
  font-size: 1.05rem;
  line-height: 1.6;

  resize: vertical;
}

.email-field input::placeholder,
.email-field textarea::placeholder {
  color: var(--drawer-muted);
}

.email-field input[readonly] {
  color: var(--drawer-muted);
}

.email-field--message {
  flex: 1;
}

/* ACTIONS */

.email-drawer__actions {
  display: flex;
  align-items: center;
  justify-content: space-between;

  gap: 2rem;

  margin-top: 1rem;
}

.email-drawer__cancel,
.email-drawer__submit {
  appearance: none;

  padding: 0;

  border: 0;
  background: none;

  font: inherit;

  cursor: pointer;
}

.email-drawer__cancel {
  color: var(--drawer-muted);

  font-size: 0.95rem;
}

.email-drawer__submit {
  display: inline-flex;
  align-items: center;

  gap: 0.75rem;

  color: var(--drawer-text);

  font-size: 1.15rem;

  transition:
    gap 180ms ease,
    opacity 180ms ease;
}

.email-drawer__submit span:last-child {
  color: var(--color-brand);
}

.email-drawer__submit:hover {
  gap: 1rem;
  opacity: 0.75;
}

/* MOBILE */

@media (max-width: 760px) {
  .email-drawer__backdrop {
    backdrop-filter: none;
    -webkit-backdrop-filter: none;
  }

  .email-drawer__panel {
    width: 100vw;

    padding: 5.5rem 1.5rem 2rem;
  }

  .email-drawer__header h2 {
    font-size: clamp(3rem, 14vw, 4.6rem);
  }

  .email-drawer__form {
    margin-top: 3rem;
  }
}
</style>