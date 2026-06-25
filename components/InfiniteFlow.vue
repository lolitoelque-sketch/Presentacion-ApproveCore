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

    <button class="menu-return" type="button" @click="goToMenu">
      menu
    </button>

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
          :class="[node.id, { active: index === activeStep, previous: index < activeStep, selected: selectedNodeId === node.id }]"
          :style="{ left: `${node.x}px`, top: `${node.y}px` }"
          role="button"
          tabindex="0"
          :aria-expanded="selectedNodeId === node.id"
          @click="selectNode(node.id)"
          @keydown.enter.prevent="selectNode(node.id)"
          @keydown.space.prevent="selectNode(node.id)"
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
        </article>

        <div
          v-if="selectedNode"
          class="node-actions"
          :style="nodeActionsStyle"
          aria-label="Opciones del nodo seleccionado"
        >
          <button
            type="button"
            :class="{ active: activePanel === 'tools' }"
            @click.stop="openNodePanel('tools')"
          >
            herramientas
          </button>
          <button
            type="button"
            :class="{ active: activePanel === 'example' }"
            @click.stop="openNodePanel('example')"
          >
            {{ selectedNode.id === 'approval' ? 'ver ruta' : 'ver ejemplo' }}
          </button>
        </div>

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

    <nav class="pager" aria-label="Navegacion del flujo">
      <button
        class="pager-btn"
        type="button"
        aria-label="Paso anterior"
        :disabled="activeStep === 0"
        @click="goPrev"
      >
        &lsaquo;
      </button>
      <div class="pager-current">
        <span>{{ currentPageLabel }}</span>
        <strong>{{ activeNode.title }}</strong>
      </div>
      <button
        class="pager-btn"
        type="button"
        aria-label="Paso siguiente"
        :disabled="activeStep === nodes.length - 1"
        @click="goNext"
      >
        &rsaquo;
      </button>
    </nav>

    <section
      v-if="panelNode"
      class="detail-layer"
      aria-live="polite"
      @click.self="closePanel"
    >
      <article
        class="detail-window"
        :class="{
          'with-image': activePanel === 'example' && panelNode.image,
          'with-route-flows': showRouteProfilePicker,
          'text-only-example': activePanel === 'example' && !panelNode.image && !showRouteProfilePicker
        }"
        role="dialog"
        :aria-label="`${panelTitle} de ${panelNode.title}`"
      >
        <button
          class="detail-close"
          type="button"
          aria-label="Cerrar panel"
          title="Cerrar"
          @click="closePanel"
        >
          X
        </button>

        <span class="detail-kicker">{{ panelTitle }}</span>
        <h2>{{ panelNode.title }}</h2>
        <p>{{ panelDescription }}</p>

        <ul v-if="activePanel === 'tools'" class="detail-list">
          <li v-for="tool in panelNode.tools" :key="tool">{{ tool }}</li>
        </ul>

        <template v-else>
          <div v-if="showRouteProfilePicker" class="route-example">
            <div class="profile-picker" aria-label="Elegir perfil para ruta de aprobacion">
              <span>Elegir perfil</span>
              <button
                v-for="profile in routeProfiles"
                :key="profile.id"
                type="button"
                :class="{ active: selectedRouteProfileId === profile.id }"
                @click="selectRouteProfile(profile.id)"
              >
                {{ profile.label }}
              </button>
            </div>

            <div v-if="selectedRouteProfile" class="route-flow-grid">
              <article
                v-for="flow in selectedRouteProfile.flows"
                :key="flow.src"
                class="route-flow-card"
              >
                <strong>{{ flow.title }}</strong>
                <figure class="example-preview route-flow-preview">
                  <img :src="flow.src" :alt="flow.alt" />
                </figure>
              </article>
            </div>

            <div v-else class="route-empty">
              Selecciona PMO u OC/IEC para ver los flujogramas.
            </div>
          </div>

          <template v-else>
            <figure v-if="panelNode.image" class="example-preview">
              <img :src="panelNode.image" :alt="panelNode.imageAlt" />
            </figure>

            <div class="detail-note">
              <span>Ejemplo</span>
              <strong>{{ panelNode.example }}</strong>
            </div>
          </template>
        </template>
      </article>
    </section>
  </main>
</template>

<script setup>
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'

const nodes = [
  {
    id: 'home',
    type: 'Inicio',
    title: 'Inicio',
    text: 'Navegacion por la app, opciones de uso y accesos principales del menu.',
    detail: 'Esta pantalla orienta al usuario antes de iniciar el flujo: que puede hacer, desde que menu entrar y como identificar sus opciones por perfil.',
    tools: ['Menu principal: punto de entrada a solicitudes, historial, dashboards y mantenimiento.', 'Opciones por perfil: cada usuario ve acciones segun su rol operativo.', 'Navegacion: regresar al menu, abrir bandejas, consultar tickets y continuar procesos pendientes.'],
    example: 'El usuario identifica si debe crear un ticket, consultar historial, revisar carritos, entrar al dashboard o ejecutar una accion de aprobacion.',
    image: '/guide-screens/01_Portada_guia_visual.png',
    imageAlt: 'Pantalla de inicio y navegacion principal de ApproveCore',
    x: 170,
    y: 570
  },
  {
    id: 'ticket',
    type: 'Registro',
    title: 'Generar ticket',
    text: 'Pantalla para crear la solicitud con datos completos antes de enviarla a aprobacion.',
    detail: 'El formulario debe explicar que se registra en cada campo para que el ticket nazca claro, trazable y listo para revision.',
    tools: ['Proyecto y etapa: ubican la solicitud dentro de la iniciativa correcta.', 'Tipo de solicitud, categoria y concepto: clasifican la necesidad para el flujo.', 'Moneda, monto y cantidad: permiten validar impacto presupuestal.', 'TDP y sustentos: conectan el ticket con la validacion financiera y la evidencia requerida.'],
    example: 'La imagen muestra un ticket con campos llenados: proyecto, etapa, tipo de solicitud, categoria, moneda, monto, concepto, cantidad, TDP, adjuntos y comentario de sustento.',
    image: '/guide-screens/04_Generar_nuevos_tickets.png',
    imageAlt: 'Pantalla de generar ticket con datos principales llenados',
    x: 500,
    y: 390
  },
  {
    id: 'required-data',
    type: 'Control',
    title: 'Datos necesarios',
    text: 'Datos y adjuntos que reducen observaciones y ayudan a que los jefes aprueben sin reprocesos.',
    detail: 'Aqui no se muestra imagen; se explican los criterios minimos para que el aprobador pueda revisar con confianza.',
    tools: ['Sustento de la necesidad: motivo claro, alcance, urgencia y beneficio esperado.', 'Datos presupuestales: proyecto, etapa, actividad, subactividad, moneda, monto y TDP.', 'Evidencia adjunta: cotizacion, requerimiento, correo de respaldo, formato o archivo tecnico cuando aplique.', 'Coherencia del pedido: el concepto, la cantidad y el monto deben coincidir con los sustentos.'],
    example: 'Antes de enviar, el solicitante valida que la descripcion sea clara, los adjuntos sean legibles y los campos financieros coincidan con la necesidad real.',
    x: 830,
    y: 560
  },
  {
    id: 'approval',
    type: 'Rutas',
    title: 'Rutas',
    text: 'Rutas de aprobacion para los perfiles PMO y OC/IEC.',
    detail: 'La app dirige el ticket segun perfil, flujo, area y rol; el usuario puede elegir el perfil para ver los flujogramas correspondientes.',
    tools: ['Perfil PMO: muestra el flujo de aprobacion TAs/ERCs y el flujo de aprobacion PMO.', 'Perfil OC/IEC: muestra el flujo OC-IEC y el flujo TAs/ERCs para OCIEC.', 'Trazabilidad: cada aprobacion, observacion o cambio de estado queda asociado al ticket.'],
    example: 'Selecciona PMO u OC/IEC para revisar las rutas completas de aprobacion.',
    x: 1160,
    y: 300
  },
  {
    id: 'tdp',
    type: 'Validacion',
    title: 'TDP',
    text: 'Validadores revisan datos clave antes de que un asignador apruebe o libere el proceso.',
    detail: 'Aqui no se muestra imagen; se explica que debe estar establecido antes de la validacion del asignador.',
    tools: ['Validadores: revisan TDP, centro de costo, categoria, monto, moneda y consistencia presupuestal.', 'Condiciones previas: ticket aprobado por la ruta funcional, sustentos completos y datos financieros coherentes.', 'Decision del asignador: confirmar TDP, corregir informacion o devolver el ticket con observacion registrada.'],
    example: 'Antes de validar, el asignador debe encontrar una solicitud aprobada, con TDP definido, presupuesto identificable y evidencia suficiente para sustentar la asignacion.',
    x: 1200,
    y: 720
  },
  {
    id: 'cart',
    type: 'Ejecucion',
    title: 'Carritos',
    text: 'Dashboard de carritos para controlar el avance operativo y financiero asociado al ticket.',
    detail: 'La pantalla permite revisar que tickets ya tienen carrito, cuales estan pendientes y que datos deben completarse para continuar.',
    tools: ['Numero de ticket y estado: ubican cada solicitud dentro del seguimiento.', 'Carrito, TA/ERC o correlativo: identifican el avance en SAP o proceso operativo.', 'Proyecto, monto y responsable: permiten priorizar y hacer seguimiento.', 'Fechas y observaciones: muestran antiguedad, bloqueos y acciones pendientes.'],
    example: 'El dashboard de carritos ayuda a revisar tickets con carrito pendiente, registros incompletos, estados financieros y responsables actuales.',
    image: '/guide-screens/05_Dashboard_carritos.png',
    imageAlt: 'Dashboard real de carritos y consulta operativa',
    x: 1530,
    y: 500
  },
  {
    id: 'dashboard',
    type: 'Control',
    title: 'Dashboard',
    text: 'Dashboard para consultar avance, presupuesto y estados importantes del proceso.',
    detail: 'El tablero ayuda a navegar informacion consolidada y entender donde esta cada solicitud dentro del control presupuestal.',
    tools: ['Filtros: proyecto, etapa, actividad, subactividad, estado y responsable.', 'Indicadores: linea base, estimado, asignado, usado y disponible.', 'Seguimiento: carritos, PO, tickets observados, aprobados o pendientes.', 'Lectura gerencial: permite detectar desviaciones, bloqueos y necesidades de accion.'],
    example: 'El dashboard resume el estado del presupuesto y del seguimiento, para revisar carritos, PO, disponibilidad y avance por proyecto.',
    image: '/guide-screens/06_Dashboard_presupuesto_tarjetero.png',
    imageAlt: 'Pantalla real de dashboard presupuestal por proyecto y actividad',
    x: 1840,
    y: 300
  },
  {
    id: 'close',
    type: 'Cierre',
    title: 'Cierre',
    text: 'El ticket finaliza cuando queda registrado el resultado y la evidencia necesaria para consulta posterior.',
    detail: 'Aqui no se muestra imagen; se explica que informacion debe quedar cerrada y trazable al final del proceso.',
    tools: ['Resultado final: aprobado, ejecutado, cancelado o cerrado segun corresponda.', 'Evidencia de cierre: carrito, PO, TA/ERC, comentario final o sustento operativo.', 'Trazabilidad: responsable, fecha, historial de acciones y observaciones.', 'Consulta posterior: el ticket queda disponible para auditoria y seguimiento.'],
    example: 'El cierre confirma que no quedan acciones pendientes y que el ticket tiene estado final, responsable, fecha y evidencia suficiente.',
    x: 2160,
    y: 590
  }
]

const paths = [
  {
    id: 'p1',
    kind: 'flow',
    to: 'ticket',
    d: 'M420 625 C455 600 460 455 500 445'
  },
  {
    id: 'p2',
    kind: 'flow',
    to: 'required-data',
    d: 'M750 445 C805 445 790 610 830 615'
  },
  {
    id: 'p3',
    kind: 'flow',
    to: 'approval',
    d: 'M1080 605 C1125 515 1115 360 1160 355'
  },
  {
    id: 'p4',
    kind: 'flow',
    to: 'tdp',
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
    to: 'dashboard',
    d: 'M1780 550 C1840 505 1800 360 1840 355'
  },
  {
    id: 'p7',
    kind: 'generate',
    to: 'close',
    d: 'M1950 355 C2070 410 2090 560 2160 625'
  }
]

const routeProfiles = [
  {
    id: 'pmo',
    label: 'PMO',
    flows: [
      {
        title: 'Flujograma de aprobacion TAs y ERCs',
        src: '/approval-flows/tas-ercs-pmo.png',
        alt: 'Flujograma de aprobacion TAs y ERCs para PMO'
      },
      {
        title: 'Flujograma de aprobacion PMO',
        src: '/approval-flows/pmo.png',
        alt: 'Flujograma de aprobacion PMO'
      }
    ]
  },
  {
    id: 'ociec',
    label: 'OC/IEC',
    flows: [
      {
        title: 'Flujograma de aprobacion OC-IEC',
        src: '/approval-flows/oc-iec.png',
        alt: 'Flujograma de aprobacion OC-IEC'
      },
      {
        title: 'Flujograma de aprobacion TAs y ERCs para OCIEC',
        src: '/approval-flows/tas-ercs-ociec.png',
        alt: 'Flujograma de aprobacion TAs y ERCs para OCIEC'
      }
    ]
  }
]

const cameraPositions = [
  { x: 610, y: 80, rotate: -1 },
  { x: 360, y: 230, rotate: 0.4 },
  { x: 110, y: 70, rotate: -0.6 },
  { x: -160, y: 270, rotate: 0.7 },
  { x: -250, y: -110, rotate: -0.4 },
  { x: -530, y: 105, rotate: 0.7 },
  { x: -730, y: 290, rotate: 0.5 },
  { x: -1050, y: 40, rotate: -0.5 }
]

const canvasScale = 0.68

const activeStep = ref(0)
const selectedNodeId = ref(null)
const focusedNodeId = ref(null)
const activePanel = ref(null)
const selectedRouteProfileId = ref(null)

const selectedNode = computed(() => {
  return nodes.find((node) => node.id === selectedNodeId.value)
})

const activeNode = computed(() => {
  return nodes[activeStep.value] || nodes[0]
})

const panelNode = computed(() => {
  if (!activePanel.value) {
    return null
  }

  return selectedNode.value
})

const panelTitle = computed(() => {
  if (activePanel.value === 'tools') {
    return 'Herramientas'
  }

  return panelNode.value?.id === 'approval' ? 'Ver ruta' : 'Ver ejemplo'
})

const panelDescription = computed(() => {
  if (!panelNode.value) {
    return ''
  }

  return activePanel.value === 'tools' ? panelNode.value.detail : panelNode.value.text
})

const showRouteProfilePicker = computed(() => {
  return activePanel.value === 'example' && panelNode.value?.id === 'approval'
})

const selectedRouteProfile = computed(() => {
  return routeProfiles.find((profile) => profile.id === selectedRouteProfileId.value)
})

const currentPageLabel = computed(() => {
  return `${String(activeStep.value + 1).padStart(2, '0')} / ${String(nodes.length).padStart(2, '0')}`
})

const nodeActionsStyle = computed(() => {
  if (!selectedNode.value) {
    return {}
  }

  return {
    left: `${selectedNode.value.x + 20}px`,
    top: `${selectedNode.value.y + 148}px`
  }
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
  activePanel.value = null
  selectedRouteProfileId.value = null
}

const goNext = () => {
  goToStep(activeStep.value + 1)
}

const goPrev = () => {
  goToStep(activeStep.value - 1)
}

const handleKeyboardNavigation = (event) => {
  if (event.key === 'Escape' && activePanel.value) {
    event.preventDefault()
    event.stopPropagation()
    closePanel()
    return
  }

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

const selectNode = (nodeId) => {
  const nodeIndex = nodes.findIndex((node) => node.id === nodeId)
  const isSelected = selectedNodeId.value === nodeId

  selectedNodeId.value = isSelected ? null : nodeId
  focusedNodeId.value = isSelected ? null : nodeId
  activePanel.value = null
  selectedRouteProfileId.value = null

  if (nodeIndex !== -1) {
    activeStep.value = nodeIndex
  }
}

const openNodePanel = (panel) => {
  activePanel.value = activePanel.value === panel ? null : panel

  if (selectedNode.value?.id === 'approval' && activePanel.value === 'example') {
    selectedRouteProfileId.value = null
  }
}

const closePanel = () => {
  activePanel.value = null
}

const selectRouteProfile = (profileId) => {
  selectedRouteProfileId.value = profileId
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

.node.selected {
  border-color: #1f6f95;
  background: rgba(221, 232, 236, 0.96);
  box-shadow:
    0 20px 34px rgba(31, 111, 149, 0.2),
    0 0 0 7px rgba(31, 111, 149, 0.12);
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

.node-actions {
  position: absolute;
  z-index: 6;
  display: grid;
  gap: 14px;
  min-width: 220px;
  padding: 4px 0;
}

.node-actions button {
  width: max-content;
  padding: 0;
  border: 0;
  color: #7f766d;
  background: transparent;
  cursor: pointer;
  font: inherit;
  font-size: 24px;
  font-weight: 900;
  line-height: 1;
  letter-spacing: 0;
  text-align: left;
}

.node-actions button:hover,
.node-actions button:focus-visible,
.node-actions button.active {
  color: #1f6f95;
}

.node-actions button:focus-visible {
  outline: 2px solid rgba(31, 111, 149, 0.72);
  outline-offset: 5px;
  border-radius: 4px;
}

.home,
.ticket,
.required-data,
.dashboard,
.close {
  border-color: #c8b8a8;
}

.approval,
.tdp,
.cart {
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

.pager-btn {
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
  transition: color 220ms ease, border-color 220ms ease, background 220ms ease;
}

.pager-btn:hover,
.pager-btn:focus-visible {
  color: #1f6f95;
  border-color: #1f6f95;
  background: #fdfaf4;
}

.pager-btn:disabled {
  cursor: default;
  opacity: 0.42;
}

.pager-current {
  display: grid;
  gap: 4px;
  min-width: 0;
}

.pager-current span {
  color: #b8a99b;
  font-size: 14px;
  font-weight: 900;
  line-height: 1;
  letter-spacing: 0.08em;
}

.pager-current strong {
  color: #7c2d12;
  font-size: 17px;
  font-weight: 900;
  line-height: 1;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.detail-layer {
  position: fixed;
  inset: 0;
  z-index: 30;
  display: grid;
  align-items: center;
  justify-items: center;
  padding: clamp(18px, 3vw, 36px);
  pointer-events: none;
}

.detail-window {
  position: relative;
  width: clamp(320px, 30vw, 460px);
  max-height: min(540px, 74vh);
  padding: 22px 26px;
  border: 1px solid #d7cabd;
  border-radius: 6px;
  color: #4f3e32;
  background: rgba(250, 247, 241, 0.98);
  box-shadow: 0 28px 74px rgba(92, 67, 49, 0.22);
  overflow: auto;
  pointer-events: auto;
}

.detail-window.with-image {
  width: clamp(520px, 54vw, 760px);
  padding: 20px 24px 22px;
}

.detail-window.text-only-example {
  width: clamp(360px, 38vw, 560px);
}

.detail-window.with-route-flows {
  width: clamp(620px, 68vw, 960px);
  max-height: min(760px, 88vh);
}

.detail-close {
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
  font-size: 16px;
  font-weight: 900;
  line-height: 1;
  z-index: 2;
}

.detail-close:hover,
.detail-close:focus-visible {
  color: #ffffff;
  background: #7c2d12;
  border-color: #7c2d12;
}

.detail-kicker {
  display: block;
  margin: 0 46px 9px 0;
  color: #8d7b6d;
  font-size: 11px;
  font-weight: 900;
  letter-spacing: 0.18em;
  text-transform: uppercase;
}

.detail-window h2 {
  margin: 0;
  color: #7c2d12;
  font-size: clamp(20px, 1.8vw, 28px);
  font-weight: 900;
  line-height: 1;
  letter-spacing: 0;
}

.detail-window p {
  margin: 10px 0 0;
  color: #4f3e32;
  font-size: clamp(13px, 1vw, 16px);
  font-weight: 700;
  line-height: 1.45;
}

.detail-list {
  display: grid;
  gap: 12px;
  margin: 18px 0 0;
  padding: 0;
  list-style: none;
}

.detail-list li {
  padding: 14px 16px;
  border-left: 4px solid #1f6f95;
  border-radius: 4px;
  background: rgba(221, 232, 236, 0.72);
  color: #4f3e32;
  font-size: 16px;
  font-weight: 800;
  line-height: 1.35;
}

.route-example {
  display: grid;
  gap: 14px;
  margin-top: 16px;
}

.profile-picker {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 10px;
  padding: 12px;
  border: 1px solid #d7cabd;
  border-radius: 6px;
  background: rgba(221, 232, 236, 0.5);
}

.profile-picker span {
  margin-right: 6px;
  color: #8d7b6d;
  font-size: 12px;
  font-weight: 900;
  letter-spacing: 0.12em;
  text-transform: uppercase;
}

.profile-picker button {
  min-width: 92px;
  padding: 9px 12px;
  border: 1px solid #c8b8a8;
  border-radius: 5px;
  color: #7c2d12;
  background: rgba(250, 247, 241, 0.92);
  cursor: pointer;
  font: inherit;
  font-size: 14px;
  font-weight: 900;
  line-height: 1;
}

.profile-picker button:hover,
.profile-picker button:focus-visible,
.profile-picker button.active {
  color: #ffffff;
  border-color: #1f6f95;
  background: #1f6f95;
}

.route-flow-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 14px;
}

.route-flow-card {
  min-width: 0;
}

.route-flow-card strong {
  display: block;
  margin-bottom: 8px;
  color: #7c2d12;
  font-size: 14px;
  font-weight: 900;
  line-height: 1.2;
}

.route-empty {
  display: grid;
  place-items: center;
  min-height: 190px;
  border: 1px dashed rgba(31, 111, 149, 0.42);
  border-radius: 6px;
  color: #7f766d;
  background: rgba(250, 247, 241, 0.72);
  font-size: 16px;
  font-weight: 800;
  text-align: center;
}

.example-preview {
  margin: 14px 0 0;
  border: 1px solid rgba(31, 111, 149, 0.28);
  border-radius: 6px;
  background: #061326;
  box-shadow: 0 14px 30px rgba(31, 111, 149, 0.16);
  overflow: hidden;
}

.example-preview img {
  display: block;
  width: 100%;
  aspect-ratio: 16 / 9;
  max-height: min(34vh, 300px);
  object-fit: cover;
  transform: scale(1.08);
  transform-origin: center center;
}

.route-flow-preview {
  margin-top: 0;
}

.route-flow-preview img {
  height: min(40vh, 360px);
  max-height: none;
  object-fit: contain;
  transform: none;
}

.detail-note {
  margin-top: 12px;
  padding: 12px 14px;
  border: 1px solid #d7cabd;
  border-radius: 4px;
  background: rgba(221, 232, 236, 0.72);
}

.detail-note span {
  display: block;
  margin-bottom: 6px;
  color: #1f6f95;
  font-size: 11px;
  font-weight: 900;
  text-transform: uppercase;
}

.detail-note strong {
  display: block;
  color: #4f3e32;
  font-size: clamp(13px, 1vw, 16px);
  line-height: 1.4;
}

@media (max-width: 800px) {
  .progress {
    left: 24px;
    right: auto;
    top: auto;
    bottom: 118px;
  }

  .camera {
    transition-duration: 900ms;
  }

  .node-actions {
    min-width: 190px;
  }

  .node-actions button {
    font-size: 21px;
  }

  .pager {
    right: 16px;
    bottom: 18px;
    grid-template-columns: 34px minmax(118px, 180px) 34px;
    gap: 8px;
    width: min(88vw, 300px);
    min-height: 54px;
    padding: 9px 10px;
  }

  .pager-btn {
    width: 30px;
    height: 30px;
    font-size: 24px;
  }

  .pager-current span {
    font-size: 12px;
  }

  .pager-current strong {
    font-size: 15px;
  }

  .detail-layer {
    align-items: center;
    justify-items: center;
    padding: 20px;
  }

  .detail-window {
    width: min(86vw, 420px);
    padding: 24px 20px;
  }

  .detail-window.with-image {
    width: min(90vw, 520px);
  }

  .detail-window.with-route-flows {
    width: min(94vw, 620px);
  }

  .route-flow-grid {
    grid-template-columns: 1fr;
  }

  .route-flow-preview img {
    height: min(42vh, 340px);
  }

  .detail-close {
    right: 14px;
    top: 12px;
    width: 30px;
    height: 30px;
  }

  .example-preview img {
    max-height: 30vh;
  }
}
</style>
