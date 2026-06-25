<template>
  <main class="hub">
    <section class="hub-content">
      <span class="hub-kicker">APPRUVECORE</span>
      <h1>Precentacion Approve Core</h1>
      <p>Elige el recorrido que quieres abrir.</p>

      <div class="hub-options">
        <button type="button" class="hub-option" @click="goToSlide(2)">
          <span>01</span>
          <strong>Presentacion flujo interactivo</strong>
          <small>Contexto del problema y paso hacia tickets trazables.</small>
        </button>

        <button type="button" class="hub-option" @click="goToSlide(3)">
          <span>02</span>
          <strong>Manual interactivo</strong>
          <small>Flujo operativo actual con pantallas, rutas y ejemplos.</small>
        </button>
      </div>
    </section>
  </main>
</template>

<script setup>
const goToSlide = (slideNumber) => {
  if (window.$slidev?.nav?.go) {
    window.$slidev.nav.go(slideNumber)
    return
  }

  const basePath = window.location.pathname.replace(/\/\d+\/?$/, '/')
  window.location.assign(`${basePath}${slideNumber}`)
}
</script>

<style scoped>
.hub {
  position: fixed;
  inset: 0;
  display: grid;
  align-items: center;
  padding: clamp(42px, 7vh, 78px) clamp(34px, 6vw, 86px);
  color: #4f3e32;
  background:
    linear-gradient(rgba(126, 93, 67, 0.075) 1px, transparent 1px),
    linear-gradient(90deg, rgba(126, 93, 67, 0.075) 1px, transparent 1px),
    radial-gradient(circle, rgba(116, 88, 68, 0.18) 1px, transparent 1.4px),
    linear-gradient(180deg, #f8f4ed 0%, #eee8df 100%);
  background-size: 72px 72px, 72px 72px, 24px 24px, auto;
  font-family: "Cascadia Code", "JetBrains Mono", "Fira Code", Consolas, ui-monospace, monospace;
}

.hub::before {
  content: "";
  position: absolute;
  inset: 0;
  background: linear-gradient(180deg, rgba(244, 240, 232, 0), rgba(227, 234, 232, 0.7));
  pointer-events: none;
}

.hub-content {
  position: relative;
  display: grid;
  gap: 22px;
  max-width: 980px;
  transform: translateY(-4vh);
}

.hub-kicker {
  color: #7c2d12;
  font-size: 15px;
  font-weight: 900;
  letter-spacing: 0.32em;
  text-transform: uppercase;
}

.hub h1 {
  margin: 0;
  color: #7c2d12;
  font-size: clamp(40px, 5.4vw, 78px);
  font-weight: 900;
  line-height: 1;
  letter-spacing: 0;
}

.hub p {
  margin: 0;
  color: #4f3e32;
  font-size: clamp(18px, 1.8vw, 24px);
  font-weight: 800;
}

.hub-options {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 18px;
  margin-top: 4px;
}

.hub-option {
  display: grid;
  min-height: 168px;
  padding: 18px 20px;
  border: 1px solid #d7cabd;
  border-radius: 6px;
  color: #4f3e32;
  background: rgba(250, 247, 241, 0.9);
  box-shadow: 0 18px 38px rgba(92, 67, 49, 0.1);
  cursor: pointer;
  font: inherit;
  text-align: left;
  transition: transform 260ms ease, border-color 260ms ease, box-shadow 260ms ease;
}

.hub-option:hover,
.hub-option:focus-visible {
  transform: translateY(-8px);
  border-color: #1f6f95;
  box-shadow:
    0 24px 44px rgba(92, 67, 49, 0.14),
    0 0 0 5px rgba(31, 111, 149, 0.1);
}

.hub-option span {
  display: grid;
  place-items: center;
  width: 31px;
  height: 31px;
  border-radius: 999px;
  color: #fff8ef;
  background: #7c2d12;
  font-weight: 900;
}

.hub-option strong {
  align-self: end;
  color: #7c2d12;
  font-size: clamp(21px, 2.1vw, 29px);
  font-weight: 900;
  line-height: 1.05;
}

.hub-option small {
  color: #7f766d;
  font-size: 13px;
  font-weight: 800;
  line-height: 1.35;
}

@media (max-width: 820px) {
  .hub-options {
    grid-template-columns: 1fr;
  }
}
</style>
