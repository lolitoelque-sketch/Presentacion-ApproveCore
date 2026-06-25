<template>
  <main class="context-flow">
    <div class="progress" aria-hidden="true">
      <span
        v-for="(item, index) in nodes"
        :key="item.id"
        :class="{ active: index === activeStep, done: index < activeStep }"
      />
    </div>

    <button class="menu-return" type="button" @click="goToMenu">
      menu
    </button>

    <section class="canvas-shell">
      <div class="camera" :style="cameraStyle">
        <svg class="paths" viewBox="0 0 2200 1100" aria-hidden="true">
          <defs>
            <marker
              id="context-arrow"
              markerWidth="8"
              markerHeight="8"
              refX="7"
              refY="3"
              orient="auto"
              markerUnits="strokeWidth"
            >
              <path d="M0,0 L0,6 L8,3 z" fill="#1f6f95" />
            </marker>
          </defs>

          <path
            v-for="path in paths"
            :key="path.id"
            class="flow-path"
            :class="{ lit: pathLit(path.to) }"
            :d="path.d"
            marker-end="url(#context-arrow)"
          />
        </svg>

        <article
          v-for="(node, index) in nodes"
          :key="node.id"
          class="node"
          :class="{ active: index === activeStep, previous: index < activeStep }"
          :style="{ left: `${node.x}px`, top: `${node.y}px` }"
          role="button"
          tabindex="0"
          :aria-expanded="selectedNodeIndex === index"
          @click="openNodeDetail(index)"
          @keydown.enter.prevent="openNodeDetail(index)"
          @keydown.space.prevent="openNodeDetail(index)"
        >
          <div class="node-topline">
            <span class="node-index">{{ String(index + 1).padStart(2, '0') }}</span>
            <span>{{ node.type }}</span>
          </div>
          <strong>{{ node.title }}</strong>
          <p>{{ node.short }}</p>
        </article>

        <div class="canvas-label label-a">Situacion inicial</div>
        <div class="canvas-label label-b">Riesgos visibles</div>
        <div class="canvas-label label-c">Camino objetivo</div>
      </div>
    </section>

    <section
      v-if="selectedNode"
      class="modal-layer"
      aria-live="polite"
      @click.self="closeNodeDetail"
    >
      <article
        class="modal-window"
        role="dialog"
        :aria-label="`Detalle de ${selectedNode.title}`"
      >
        <button
          class="modal-close"
          type="button"
          aria-label="Cerrar detalle"
          title="Cerrar"
          @click="closeNodeDetail"
        >
          X
        </button>

        <span>{{ selectedNode.type }}</span>
        <h2>{{ selectedNode.title }}</h2>
        <p>{{ selectedNode.detail }}</p>

        <div class="modal-note">
          <small>Impacto</small>
          <strong>{{ selectedNode.result }}</strong>
        </div>
      </article>
    </section>

    <nav class="pager" aria-label="Navegacion del contexto">
      <button type="button" :disabled="activeStep === 0" @click="goPrev">&lsaquo;</button>
      <div>
        <span>{{ currentPageLabel }}</span>
        <strong>{{ activeNode.title }}</strong>
      </div>
      <button type="button" :disabled="activeStep === nodes.length - 1" @click="goNext">&rsaquo;</button>
    </nav>
  </main>
</template>

<script setup>
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'

const nodes = [
  {
    id: 'context',
    type: 'Contexto',
    title: 'Proceso manual',
    short: 'Correos, adjuntos y seguimiento individual.',
    detail: 'Las solicitudes se gestionaban con correos, archivos adjuntos y seguimiento individual.',
    result: 'El estado real dependia de revisar conversaciones y coordinaciones dispersas.',
    x: 190,
    y: 470
  },
  {
    id: 'visibility',
    type: 'Riesgo',
    title: 'Visibilidad limitada',
    short: 'No habia bandeja unica.',
    detail: 'No existia una bandeja unica para consultar el avance de cada solicitud.',
    result: 'Era dificil saber si algo estaba pendiente, aprobado, observado o detenido.',
    x: 520,
    y: 260
  },
  {
    id: 'owners',
    type: 'Operacion',
    title: 'Responsables dispersos',
    short: 'La accion dependia de cadenas de correo.',
    detail: 'La siguiente accion dependia de cadenas de correo y coordinaciones fuera del sistema.',
    result: 'El responsable actual no siempre era evidente para quien consultaba el caso.',
    x: 850,
    y: 560
  },
  {
    id: 'finance',
    type: 'Control',
    title: 'Control financiero fragil',
    short: 'Montos, carritos, PO y estados en varias fuentes.',
    detail: 'Montos, carritos, PO y estados podian quedar repartidos en varias fuentes.',
    result: 'La lectura presupuestal necesitaba consolidacion y trazabilidad.',
    x: 1200,
    y: 300
  },
  {
    id: 'ticket',
    type: 'Solucion',
    title: 'Ticket trazable',
    short: 'Una solicitud visible de inicio a fin.',
    detail: 'ApproveCore convierte la coordinacion en un ticket estructurado, medible y consultable.',
    result: 'Cada solicitud puede mostrar estado, responsable, observaciones, presupuesto y cierre.',
    x: 1540,
    y: 560
  },
  {
    id: 'control',
    type: 'Resultado',
    title: 'Flujo controlado',
    short: 'Menos correos, mas seguimiento.',
    detail: 'El proceso pasa de coordinaciones manuales a una ruta visible con acciones por rol.',
    result: 'La organizacion gana trazabilidad, control operativo y lectura presupuestal.',
    x: 1800,
    y: 270
  }
]

const paths = [
  { id: 'p1', to: 'visibility', d: 'M430 525 C495 470 465 320 520 315' },
  { id: 'p2', to: 'owners', d: 'M760 315 C850 340 795 610 850 615' },
  { id: 'p3', to: 'finance', d: 'M1090 615 C1165 570 1120 360 1200 355' },
  { id: 'p4', to: 'ticket', d: 'M1445 355 C1540 410 1480 610 1540 615' },
  { id: 'p5', to: 'control', d: 'M1745 615 C1840 570 1710 320 1800 325' }
]

const cameraPositions = [
  { x: 470, y: 60, rotate: -0.8 },
  { x: 250, y: 220, rotate: 0.5 },
  { x: -40, y: 30, rotate: -0.4 },
  { x: -310, y: 210, rotate: 0.5 },
  { x: -580, y: 10, rotate: -0.4 },
  { x: -780, y: 250, rotate: 0.5 }
]

const canvasScale = 0.72
const activeStep = ref(0)
const selectedNodeIndex = ref(null)

const activeNode = computed(() => nodes[activeStep.value] || nodes[0])

const selectedNode = computed(() => {
  if (selectedNodeIndex.value === null) {
    return null
  }

  return nodes[selectedNodeIndex.value]
})

const currentPageLabel = computed(() => {
  return `${String(activeStep.value + 1).padStart(2, '0')} / ${String(nodes.length).padStart(2, '0')}`
})

const cameraStyle = computed(() => {
  const position = cameraPositions[activeStep.value] || cameraPositions[0]

  return {
    transform: `translate(-50%, -50%) translate3d(${position.x}px, ${position.y}px, 0) scale(${canvasScale}) rotate(${position.rotate}deg)`
  }
})

const goToStep = (step) => {
  activeStep.value = Math.min(Math.max(step, 0), nodes.length - 1)
  selectedNodeIndex.value = null
}

const goNext = () => {
  goToStep(activeStep.value + 1)
}

const goPrev = () => {
  goToStep(activeStep.value - 1)
}

const handleKeyboardNavigation = (event) => {
  if (event.key === 'Escape' && selectedNode.value) {
    event.preventDefault()
    event.stopPropagation()
    closeNodeDetail()
    return
  }

  const nextKeys = ['ArrowRight', 'ArrowDown', 'PageDown', ' ']
  const prevKeys = ['ArrowLeft', 'ArrowUp', 'PageUp']

  if (![...nextKeys, ...prevKeys].includes(event.key)) {
    return
  }

  event.preventDefault()

  if (nextKeys.includes(event.key)) {
    goNext()
    return
  }

  goPrev()
}

onMounted(() => {
  window.addEventListener('keydown', handleKeyboardNavigation, true)
})

onBeforeUnmount(() => {
  window.removeEventListener('keydown', handleKeyboardNavigation, true)
})

const openNodeDetail = (index) => {
  activeStep.value = index
  selectedNodeIndex.value = index
}

const closeNodeDetail = () => {
  selectedNodeIndex.value = null
}

const goToMenu = () => {
  if (window.$slidev?.nav?.go) {
    window.$slidev.nav.go(1)
    return
  }

  const basePath = window.location.pathname.replace(/\/\d+\/?$/, '/')
  window.location.assign(basePath)
}

const pathLit = (targetId) => {
  const targetIndex = nodes.findIndex((node) => node.id === targetId)
  return targetIndex <= activeStep.value
}
</script>

<style scoped>
.context-flow {
  position: fixed;
  inset: 0;
  overflow: hidden;
  color: #4f3e32;
  background:
    linear-gradient(rgba(126, 93, 67, 0.075) 1px, transparent 1px),
    linear-gradient(90deg, rgba(126, 93, 67, 0.075) 1px, transparent 1px),
    radial-gradient(circle, rgba(116, 88, 68, 0.18) 1px, transparent 1.4px),
    linear-gradient(180deg, #f8f4ed 0%, #eee8df 100%);
  background-size: 72px 72px, 72px 72px, 24px 24px, auto;
  font-family: "Cascadia Code", "JetBrains Mono", "Fira Code", Consolas, ui-monospace, monospace;
}

.context-flow::before {
  content: "";
  position: absolute;
  inset: 0;
  background: linear-gradient(180deg, rgba(244, 240, 232, 0), rgba(227, 234, 232, 0.76));
  pointer-events: none;
}

.progress {
  position: absolute;
  right: clamp(28px, 5vw, 72px);
  top: clamp(34px, 6vh, 72px);
  display: flex;
  gap: 10px;
  z-index: 12;
}

.progress span {
  width: 12px;
  height: 12px;
  border: 1px solid rgba(124, 45, 18, 0.28);
  border-radius: 999px;
  background: rgba(250, 247, 241, 0.86);
}

.progress span.done,
.progress span.active {
  border-color: #1f6f95;
  background: #1f6f95;
}

.progress span.active {
  width: 34px;
}

.menu-return {
  position: fixed;
  left: clamp(22px, 3vw, 40px);
  top: clamp(24px, 4vh, 44px);
  z-index: 26;
  padding: 10px 14px;
  border: 1px solid #d7cabd;
  border-radius: 6px;
  color: #7c2d12;
  background: rgba(250, 247, 241, 0.92);
  box-shadow: 0 14px 30px rgba(92, 67, 49, 0.1);
  cursor: pointer;
  font: inherit;
  font-size: 13px;
  font-weight: 900;
  letter-spacing: 0.14em;
  line-height: 1;
  text-transform: uppercase;
}

.menu-return:hover,
.menu-return:focus-visible {
  color: #ffffff;
  border-color: #1f6f95;
  background: #1f6f95;
}

.canvas-shell {
  position: absolute;
  inset: 0;
  z-index: 3;
  perspective: 1400px;
}

.camera {
  position: absolute;
  left: 50%;
  top: 50%;
  width: 2200px;
  height: 1100px;
  transform-origin: center center;
  transition: transform 1100ms cubic-bezier(0.22, 1, 0.36, 1);
}

.camera::before {
  content: "";
  position: absolute;
  inset: 0;
  background-image:
    radial-gradient(circle, rgba(116, 88, 68, 0.2) 1px, transparent 1.5px),
    linear-gradient(rgba(126, 93, 67, 0.08) 1px, transparent 1px),
    linear-gradient(90deg, rgba(126, 93, 67, 0.08) 1px, transparent 1px);
  background-size: 42px 42px, 160px 160px, 160px 160px;
}

.paths {
  position: absolute;
  inset: 0;
  z-index: 1;
  overflow: visible;
}

.flow-path {
  fill: none;
  stroke: rgba(31, 111, 149, 0.9);
  stroke-width: 3;
  stroke-linecap: round;
  stroke-dasharray: 14 18;
  animation: dash-flow 520ms linear infinite;
}

.flow-path.lit {
  stroke: #1f6f95;
  stroke-width: 3.6;
}

@keyframes dash-flow {
  to {
    stroke-dashoffset: -60;
  }
}

.node {
  position: absolute;
  z-index: 3;
  width: 292px;
  min-height: 148px;
  padding: 18px 20px;
  border: 1px solid #c8b8a8;
  border-radius: 6px;
  background: rgba(250, 247, 241, 0.9);
  box-shadow: 0 12px 22px rgba(92, 67, 49, 0.06);
  cursor: pointer;
  transition: transform 420ms ease, border-color 420ms ease, box-shadow 420ms ease;
}

.node:hover,
.node.active {
  transform: translateY(-7px);
  border-color: #1f6f95;
  background: rgba(221, 232, 236, 0.94);
  box-shadow:
    0 18px 34px rgba(31, 111, 149, 0.14),
    0 0 0 5px rgba(31, 111, 149, 0.1);
}

.node.previous {
  border-color: rgba(124, 45, 18, 0.26);
}

.node-topline {
  display: flex;
  justify-content: space-between;
  margin-bottom: 14px;
  color: #8d7b6d;
  font-size: 12px;
  font-weight: 900;
  letter-spacing: 0.18em;
  text-transform: uppercase;
}

.node-index {
  display: grid;
  place-items: center;
  width: 28px;
  height: 28px;
  border-radius: 999px;
  color: #fff8ef;
  background: #7c2d12;
  letter-spacing: 0;
}

.node strong {
  display: block;
  color: #7c2d12;
  font-size: 29px;
  font-weight: 900;
  line-height: 1;
}

.node p {
  margin: 12px 0 0;
  color: #7f766d;
  font-size: 14px;
  font-weight: 800;
  line-height: 1.35;
}

.canvas-label {
  position: absolute;
  z-index: 2;
  padding: 10px 14px;
  border: 1px solid #d7cabd;
  border-radius: 999px;
  color: #7c2d12;
  background: rgba(250, 247, 241, 0.82);
  font-size: 16px;
  font-weight: 900;
  text-transform: uppercase;
}

.label-a {
  left: 150px;
  top: 375px;
}

.label-b {
  left: 820px;
  top: 410px;
}

.label-c {
  left: 1530px;
  top: 455px;
}

.modal-layer {
  position: fixed;
  inset: 0;
  z-index: 30;
  display: grid;
  place-items: center;
  padding: clamp(18px, 3vw, 36px);
  background: rgba(79, 62, 50, 0.08);
  pointer-events: auto;
}

.modal-window {
  position: relative;
  width: clamp(360px, 40vw, 580px);
  max-height: min(560px, 78vh);
  padding: 28px 32px;
  border: 1px solid #d7cabd;
  border-radius: 6px;
  color: #4f3e32;
  background: rgba(250, 247, 241, 0.98);
  box-shadow: 0 28px 74px rgba(92, 67, 49, 0.22);
  overflow: auto;
}

.modal-close {
  position: absolute;
  right: 18px;
  top: 16px;
  display: grid;
  place-items: center;
  width: 32px;
  height: 32px;
  border: 2px solid rgba(124, 45, 18, 0.38);
  border-radius: 6px;
  color: #7c2d12;
  background: #fffaf2;
  box-shadow: 0 10px 22px rgba(92, 67, 49, 0.14);
  cursor: pointer;
  font: inherit;
  font-size: 16px;
  font-weight: 900;
  line-height: 1;
}

.modal-close:hover,
.modal-close:focus-visible {
  color: #ffffff;
  background: #7c2d12;
  border-color: #7c2d12;
}

.modal-window > span {
  display: block;
  margin-right: 46px;
  color: #8d7b6d;
  font-size: 12px;
  font-weight: 900;
  letter-spacing: 0.18em;
  text-transform: uppercase;
}

.modal-window h2 {
  margin: 12px 0 0;
  color: #7c2d12;
  font-size: clamp(28px, 3vw, 42px);
  font-weight: 900;
  line-height: 1;
}

.modal-window p {
  margin: 20px 0 0;
  color: #4f3e32;
  font-size: clamp(17px, 1.5vw, 21px);
  font-weight: 800;
  line-height: 1.42;
}

.modal-note {
  margin-top: 26px;
  padding: 18px 20px;
  border: 1px solid #d7cabd;
  border-left: 4px solid #1f6f95;
  border-radius: 4px;
  background: rgba(221, 232, 236, 0.74);
}

.modal-note small {
  display: block;
  margin-bottom: 8px;
  color: #1f6f95;
  font-size: 12px;
  font-weight: 900;
  text-transform: uppercase;
}

.modal-note strong {
  display: block;
  color: #4f3e32;
  font-size: clamp(15px, 1.3vw, 18px);
  line-height: 1.4;
}

.pager {
  position: fixed;
  right: clamp(18px, 3vw, 36px);
  bottom: clamp(22px, 4vh, 42px);
  z-index: 24;
  display: grid;
  grid-template-columns: 36px minmax(150px, 220px) 36px;
  align-items: center;
  gap: 10px;
  min-height: 58px;
  padding: 10px 12px;
  border: 1px solid #d7cabd;
  border-radius: 6px;
  background: rgba(250, 247, 241, 0.92);
  box-shadow: 0 18px 44px rgba(92, 67, 49, 0.12);
}

.pager button {
  display: grid;
  place-items: center;
  width: 32px;
  height: 32px;
  border: 2px solid #d7cabd;
  border-radius: 5px;
  color: #7f766d;
  background: rgba(255, 250, 242, 0.96);
  cursor: pointer;
  font: inherit;
  font-size: 26px;
  font-weight: 800;
  line-height: 0.8;
}

.pager button:hover,
.pager button:focus-visible {
  color: #1f6f95;
  border-color: #1f6f95;
}

.pager button:disabled {
  cursor: default;
  opacity: 0.42;
}

.pager div {
  display: grid;
  gap: 4px;
  min-width: 0;
}

.pager span {
  color: #b8a99b;
  font-size: 14px;
  font-weight: 900;
  line-height: 1;
  letter-spacing: 0.08em;
}

.pager strong {
  color: #7c2d12;
  font-size: 17px;
  font-weight: 900;
  line-height: 1;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

@media (max-width: 900px) {
  .modal-window {
    width: min(88vw, 520px);
    padding: 26px 22px;
  }
}
</style>
