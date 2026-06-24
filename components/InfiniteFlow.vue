<template>
  <main class="viewport">
    <div class="ambient ambient-a" />
    <div class="ambient ambient-b" />

    <div class="progress" aria-hidden="true">
      <span
        v-for="(item, index) in nodes"
        :key="item.id"
        :class="{ active: index === activeStep, done: index < activeStep }"
      />
    </div>

    <section class="canvas-shell">
      <div class="camera" :style="cameraStyle">
        <svg class="paths" viewBox="0 0 2600 1300" aria-hidden="true">
          <defs>
            <marker
              id="flow-arrow"
              markerWidth="8"
              markerHeight="8"
              refX="7"
              refY="3"
              orient="auto"
              markerUnits="strokeWidth"
            >
              <path d="M0,0 L0,6 L8,3 z" fill="#1f6f95" />
            </marker>

            <marker
              id="generate-arrow"
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
            :class="[path.kind, { lit: pathLit(path.to) }]"
            :d="path.d"
            :marker-end="path.kind === 'generate' ? 'url(#generate-arrow)' : 'url(#flow-arrow)'"
          />
        </svg>

        <article
          v-for="(node, index) in nodes"
          :key="node.id"
          class="node"
          :class="[node.id, { active: index === activeStep, previous: index < activeStep }]"
          :style="{ left: `${node.x}px`, top: `${node.y}px` }"
          role="button"
          tabindex="0"
          :aria-expanded="selectedNodeId === node.id"
          @click="togglePopup(node.id)"
          @keydown.enter.prevent="togglePopup(node.id)"
          @keydown.space.prevent="togglePopup(node.id)"
        >
          <div class="node-topline">
            <span class="node-index">
              {{ String(index + 1).padStart(2, '0') }}
            </span>
            <span class="node-type">
              {{ node.type }}
            </span>
          </div>

          <div class="node-title">
            {{ node.title }}
          </div>

          <span class="node-action">Ver detalle</span>
        </article>

        <div class="cluster cluster-data">
          <span>Datos maestros</span>
          <i />
          <i />
          <i />
        </div>

        <div class="cluster cluster-automation">
          <span>Seguimiento</span>
          <i />
          <i />
          <i />
          <i />
        </div>

        <div class="canvas-label label-start">Entrada</div>
        <div class="canvas-label label-core">Validacion y presupuesto</div>
        <div class="canvas-label label-output">Ejecucion y cierre</div>
      </div>
    </section>

    <section
      v-if="selectedNode"
      class="modal-layer"
      aria-live="polite"
      @click.self="closePopup"
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
          @click="closePopup"
        >
          x
        </button>

        <span class="modal-kicker">{{ selectedNode.type }}</span>
        <h2>{{ selectedNode.title }}</h2>
        <p>{{ selectedNode.detail }}</p>

        <div class="modal-note">
          <span>Resumen</span>
          <strong>{{ selectedNode.text }}</strong>
        </div>
      </article>
    </section>
  </main>
</template>

<script setup>
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'

const nodes = [
  {
    id: 'request',
    type: 'Inicio',
    title: 'Solicitud',
    text: 'El usuario crea un ticket con proyecto, monto, TDP y sustentos.',
    detail: 'AppruveCore convierte la necesidad operativa o presupuestal en un registro estructurado.',
    x: 170,
    y: 570
  },
  {
    id: 'data',
    type: 'Registro',
    title: 'Datos',
    text: 'Se registran proyecto, etapa, actividad, subactividad, moneda y asignaciones.',
    detail: 'El ticket nace con informacion suficiente para ser aprobado, corregido y auditado.',
    x: 500,
    y: 390
  },
  {
    id: 'approval',
    type: 'Aprobacion',
    title: 'Ruta',
    text: 'La app dirige el ticket al aprobador correcto segun flujo, area y rol.',
    detail: 'PMO, OC/IEC, TA/ERC y reembolsos pueden tener rutas distintas, pero conservan trazabilidad.',
    x: 830,
    y: 560
  },
  {
    id: 'observation',
    type: 'Control',
    title: 'Observacion',
    text: 'Si falta informacion, el ticket vuelve para correccion con motivo registrado.',
    detail: 'Las observaciones dejan evidencia y evitan que la correccion se pierda en correos.',
    x: 1160,
    y: 300
  },
  {
    id: 'budget',
    type: 'Asignacion',
    title: 'TDP',
    text: 'Asignadores CAPEX/OPEX validan presupuesto, centro de costo y TDP.',
    detail: 'Esta etapa conecta la aprobacion funcional con el control financiero del proyecto.',
    x: 1200,
    y: 720
  },
  {
    id: 'cart',
    type: 'Ejecucion',
    title: 'Carrito',
    text: 'Se registra carrito, TA/ERC o correlativo requerido para continuar.',
    detail: 'El proceso posterior queda asociado al ticket y al detalle financiero correspondiente.',
    x: 1530,
    y: 500
  },
  {
    id: 'po',
    type: 'Finanzas',
    title: 'PO',
    text: 'El avance financiero permite distinguir sin carrito, con carrito, con PO o cancelado.',
    detail: 'El estado financiero muestra si la solicitud aprobada ya avanzo en ejecucion presupuestal.',
    x: 1840,
    y: 780
  },
  {
    id: 'dashboard',
    type: 'Control',
    title: 'Dashboard',
    text: 'Proyecto, etapa, actividad y subactividad se consolidan para lectura presupuestal.',
    detail: 'Dashboards y tarjetero resumen linea base, estimado, carrito, PO, uso y disponible.',
    x: 1980,
    y: 300
  },
  {
    id: 'close',
    type: 'Cierre',
    title: 'Cierre',
    text: 'El ticket finaliza con responsable, fecha, historial y evidencia disponible.',
    detail: 'El cierre deja trazabilidad completa para consulta, control operativo y auditoria.',
    x: 2160,
    y: 590
  }
]

const paths = [
  {
    id: 'p1',
    kind: 'flow',
    to: 'data',
    d: 'M420 625 C455 600 460 455 500 445'
  },
  {
    id: 'p2',
    kind: 'flow',
    to: 'approval',
    d: 'M750 445 C805 445 790 610 830 615'
  },
  {
    id: 'p3',
    kind: 'flow',
    to: 'observation',
    d: 'M1080 605 C1125 515 1115 360 1160 355'
  },
  {
    id: 'p4',
    kind: 'flow',
    to: 'budget',
    d: 'M1080 640 C1130 715 1140 775 1200 775'
  },
  {
    id: 'p5',
    kind: 'generate',
    to: 'cart',
    d: 'M1450 775 C1515 735 1488 565 1530 555'
  },
  {
    id: 'p6',
    kind: 'flow',
    to: 'po',
    d: 'M1780 580 C1840 630 1795 830 1840 835'
  },
  {
    id: 'p7',
    kind: 'flow',
    to: 'dashboard',
    d: 'M1780 550 C1875 505 1880 360 1980 355'
  },
  {
    id: 'p8',
    kind: 'generate',
    to: 'close',
    d: 'M2090 835 C2180 790 2115 660 2160 650'
  },
  {
    id: 'p9',
    kind: 'flow',
    to: 'close',
    d: 'M2230 355 C2310 410 2235 580 2160 625'
  }
]

const cameraPositions = [
  { x: 610, y: 80, rotate: -1 },
  { x: 360, y: 230, rotate: 0.4 },
  { x: 110, y: 70, rotate: -0.6 },
  { x: -160, y: 270, rotate: 0.7 },
  { x: -250, y: -110, rotate: -0.4 },
  { x: -530, y: 105, rotate: 0.7 },
  { x: -780, y: -140, rotate: -0.4 },
  { x: -915, y: 290, rotate: 0.5 },
  { x: -1050, y: 40, rotate: -0.5 }
]

const canvasScale = 0.68

const activeStep = ref(0)
const selectedNodeId = ref(null)
const focusedNodeId = ref(null)

const selectedNode = computed(() => {
  return nodes.find((node) => node.id === selectedNodeId.value)
})

const cameraStep = computed(() => {
  if (!focusedNodeId.value) {
    return activeStep.value
  }

  const selectedIndex = nodes.findIndex((node) => node.id === focusedNodeId.value)

  if (selectedIndex === -1) {
    return activeStep.value
  }

  return selectedIndex
})

const cameraStyle = computed(() => {
  const position = cameraPositions[cameraStep.value] || cameraPositions[0]

  return {
    transform: `translate(-50%, -50%) translate3d(${position.x}px, ${position.y}px, 0) scale(${canvasScale}) rotate(${position.rotate}deg)`
  }
})

const goToStep = (step) => {
  const minStep = 0
  const maxStep = nodes.length - 1

  activeStep.value = Math.min(Math.max(step, minStep), maxStep)
  selectedNodeId.value = null
  focusedNodeId.value = null
}

const goNext = () => {
  goToStep(activeStep.value + 1)
}

const goPrev = () => {
  goToStep(activeStep.value - 1)
}

const handleKeyboardNavigation = (event) => {
  const nextKeys = ['ArrowRight', 'ArrowDown', 'PageDown', ' ']
  const prevKeys = ['ArrowLeft', 'ArrowUp', 'PageUp']

  if (![...nextKeys, ...prevKeys].includes(event.key)) {
    return
  }

  event.preventDefault()
  event.stopPropagation()

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

const togglePopup = (nodeId) => {
  selectedNodeId.value = selectedNodeId.value === nodeId ? null : nodeId
  focusedNodeId.value = nodeId
}

const closePopup = () => {
  selectedNodeId.value = null
}

const pathLit = (targetId) => {
  const targetIndex = nodes.findIndex((node) => node.id === targetId)
  return targetIndex <= activeStep.value
}
</script>

<style scoped>
.viewport {
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

.viewport::before {
  content: "";
  position: absolute;
  inset: 0;
  background: linear-gradient(180deg, rgba(244, 240, 232, 0), rgba(227, 234, 232, 0.76));
  pointer-events: none;
}

.ambient {
  display: none;
}

.ambient-a {
  right: 8%;
  top: 9%;
  width: 180px;
  height: 180px;
  background: rgba(125, 211, 252, 0.11);
}

.ambient-b {
  left: 10%;
  bottom: 8%;
  width: 220px;
  height: 220px;
  background: rgba(255, 178, 138, 0.09);
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
  transition: width 450ms ease, background 450ms ease, border-color 450ms ease;
}

.progress span.done,
.progress span.active {
  border-color: #1f6f95;
  background: #1f6f95;
}

.progress span.active {
  width: 34px;
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
  width: 2600px;
  height: 1300px;
  transform-origin: center center;
  transition: transform 1200ms cubic-bezier(0.22, 1, 0.36, 1);
  will-change: transform;
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
  border: 0;
  border-radius: 0;
}

.paths {
  position: absolute;
  inset: 0;
  z-index: 1;
  overflow: visible;
}

.flow-path {
  fill: none;
  stroke: rgba(31, 111, 149, 0.92);
  stroke-width: 3;
  stroke-linecap: round;
  stroke-dasharray: 14 18;
  opacity: 1;
  animation: dash-flow 520ms linear infinite;
  transition: stroke 450ms ease, stroke-width 450ms ease;
}

.flow-path.generate {
  stroke: rgba(31, 111, 149, 0.92);
  stroke-dasharray: 14 18;
  animation-duration: 520ms;
}

.flow-path.lit {
  stroke: #1f6f95;
  stroke-width: 3.6;
}

.flow-path.generate.lit {
  stroke: #1f6f95;
}

@keyframes dash-flow {
  to {
    stroke-dashoffset: -60;
  }
}

.node {
  position: absolute;
  z-index: 3;
  width: 270px;
  min-height: 132px;
  padding: 18px 20px;
  border: 1px solid #c8b8a8;
  border-radius: 6px;
  background: rgba(250, 247, 241, 0.9);
  box-shadow: 0 12px 22px rgba(92, 67, 49, 0.06);
  cursor: pointer;
  transition:
    transform 520ms ease,
    box-shadow 520ms ease,
    border-color 520ms ease,
    background 520ms ease;
}

.node:hover {
  transform: translateY(-8px) scale(1.015);
  border-color: rgba(31, 111, 149, 0.72);
  background: #fdfaf4;
  box-shadow:
    0 18px 34px rgba(92, 67, 49, 0.12),
    0 0 0 5px rgba(31, 111, 149, 0.08);
}

.node.active {
  transform: translateY(-6px);
  border-color: #2a7192;
  background: rgba(221, 232, 236, 0.9);
  box-shadow:
    0 15px 24px rgba(31, 111, 149, 0.16),
    0 0 0 5px rgba(31, 111, 149, 0.1);
}

.node.active:hover {
  transform: translateY(-10px) scale(1.02);
  border-color: #2a7192;
  background: rgba(221, 232, 236, 0.96);
  box-shadow:
    0 20px 38px rgba(31, 111, 149, 0.18),
    0 0 0 5px rgba(31, 111, 149, 0.12);
}

.node.previous {
  border-color: rgba(124, 45, 18, 0.26);
}

.node-topline {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 18px;
  color: #8d7b6d;
  font-size: 12px;
  font-weight: 800;
  letter-spacing: 0.22em;
  text-transform: uppercase;
}

.node-index {
  display: grid;
  place-items: center;
  height: 28px;
  width: 28px;
  border-radius: 999px;
  color: #fff8ef;
  background: #7c2d12;
  letter-spacing: 0;
}

.node-title {
  display: block;
  width: 100%;
  margin: 0;
  min-height: 30px;
  padding: 0;
  border: 0;
  color: #7c2d12;
  background: transparent;
  font: inherit;
  text-align: left;
  word-break: normal;
  overflow-wrap: anywhere;
  font-size: 31px;
  font-weight: 900;
  line-height: 1;
  letter-spacing: 0;
}

.node-title:hover,
.node-title:focus-visible {
  color: #1f6f95;
}

.node-title:focus-visible {
  outline: 2px solid rgba(31, 111, 149, 0.72);
  outline-offset: 6px;
  border-radius: 4px;
}

.node-action {
  display: block;
  margin-top: 14px;
  padding: 0;
  border: 0;
  border-radius: 0;
  color: #7f766d;
  background: transparent;
  font-size: 14px;
  font-weight: 800;
}

.request,
.data,
.dashboard,
.close {
  border-color: #c8b8a8;
}

.approval,
.observation,
.budget,
.cart,
.po {
  border-color: #c8b8a8;
}

.cluster {
  position: absolute;
  z-index: 2;
  display: grid;
  grid-template-columns: repeat(2, 54px);
  gap: 14px;
  padding: 20px;
  border: 1px dashed rgba(124, 45, 18, 0.22);
  border-radius: 4px;
  background: rgba(250, 247, 241, 0.38);
}

.cluster span {
  grid-column: 1 / -1;
  color: #8d7b6d;
  font-size: 16px;
  font-weight: 800;
  letter-spacing: 0.12em;
  text-transform: uppercase;
}

.cluster i {
  display: block;
  width: 54px;
  height: 54px;
  border: 1px solid rgba(124, 45, 18, 0.16);
  border-radius: 4px;
  background: rgba(31, 111, 149, 0.07);
}

.cluster-data {
  left: 1180px;
  top: 805px;
}

.cluster-automation {
  left: 1695px;
  top: 1080px;
}

.cluster-automation i {
  background: rgba(124, 45, 18, 0.06);
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
  font-weight: 800;
  letter-spacing: 0.05em;
  text-transform: uppercase;
}

.label-start {
  left: 145px;
  top: 475px;
}

.label-core {
  left: 820px;
  top: 455px;
}

.label-output {
  left: 2030px;
  top: 500px;
}

.modal-layer {
  position: fixed;
  inset: 0;
  z-index: 30;
  display: grid;
  place-items: center;
  pointer-events: none;
}

.modal-window {
  position: relative;
  width: clamp(340px, 33vw, 520px);
  max-height: min(560px, 70vh);
  padding: 44px 46px;
  border: 1px solid #d7cabd;
  border-radius: 6px;
  color: #4f3e32;
  background: rgba(250, 247, 241, 0.98);
  box-shadow: 0 34px 90px rgba(92, 67, 49, 0.24);
  pointer-events: auto;
}

.modal-close {
  position: absolute;
  right: 20px;
  top: 18px;
  display: grid;
  place-items: center;
  width: 34px;
  height: 34px;
  border: 1px solid #d7cabd;
  border-radius: 4px;
  color: #7f766d;
  background: #fffaf2;
  cursor: pointer;
  font-size: 18px;
  line-height: 1;
}

.modal-close:hover,
.modal-close:focus-visible {
  color: #ffffff;
  background: #7c2d12;
  border-color: #7c2d12;
}

.modal-kicker {
  display: block;
  margin-bottom: 22px;
  color: #8d7b6d;
  font-size: 15px;
  font-weight: 900;
  letter-spacing: 0.18em;
  text-transform: uppercase;
}

.modal-window h2 {
  margin: 0;
  color: #7c2d12;
  font-size: clamp(34px, 4vw, 52px);
  font-weight: 900;
  line-height: 1;
  letter-spacing: 0;
}

.modal-window p {
  margin: 28px 0 0;
  color: #4f3e32;
  font-size: clamp(19px, 1.8vw, 25px);
  font-weight: 700;
  line-height: 1.45;
}

.modal-note {
  margin-top: 48px;
  padding: 22px;
  border: 1px solid #d7cabd;
  border-radius: 4px;
  background: rgba(221, 232, 236, 0.72);
}

.modal-note span {
  display: block;
  margin-bottom: 12px;
  color: #1f6f95;
  font-size: 14px;
  font-weight: 900;
  text-transform: uppercase;
}

.modal-note strong {
  display: block;
  color: #4f3e32;
  font-size: clamp(17px, 1.5vw, 21px);
  line-height: 1.4;
}

@media (max-width: 800px) {
  .progress {
    left: 24px;
    right: auto;
    top: auto;
    bottom: 24px;
  }

  .camera {
    transition-duration: 900ms;
  }

  .modal-window {
    width: min(86vw, 420px);
    padding: 34px 28px;
  }
}
</style>
