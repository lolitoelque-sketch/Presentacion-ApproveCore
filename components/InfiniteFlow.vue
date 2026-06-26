<template>
  <main class="viewport">
    <div class="ambient ambient-a" />
    <div class="ambient ambient-b" />

    <div class="progress" :class="{ overview: overviewMode }" aria-hidden="true">
      <span
        v-for="(item, index) in nodes"
        :key="item.id"
        :class="{ active: index === activeStep, done: index < activeStep }"
      />
    </div>

    <button class="menu-return" type="button" @click="goToMenu">
      menu
    </button>

    <button
      class="overview-toggle"
      type="button"
      :aria-pressed="overviewMode"
      @click="toggleOverview"
    >
      {{ overviewMode ? 'volver' : 'vista completa' }}
    </button>

    <section class="canvas-shell">
      <div class="camera" :style="cameraStyle">
        <svg class="paths" viewBox="0 0 3400 2150" aria-hidden="true">
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
          :class="[node.id, node.tone, node.variant, { active: index === activeStep, previous: index < activeStep, selected: selectedNodeId === node.id }]"
          :style="{ left: `${node.x}px`, top: `${node.y}px`, width: node.width ? `${node.width}px` : undefined }"
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

          <p v-if="node.short" class="node-summary">
            {{ node.short }}
          </p>

          <ol v-if="node.steps" class="node-steps">
            <li v-for="step in node.steps" :key="step.label">
              <span>{{ step.label }}</span>
              <strong>{{ step.title }}</strong>
            </li>
          </ol>

          <div v-if="node.states" class="state-strip" aria-label="Estados del ticket">
            <span v-for="state in node.states" :key="state">{{ state }}</span>
          </div>
        </article>

        <div
          v-if="selectedNode && !overviewMode"
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

        <article
          v-for="note in notes"
          :key="note.id"
          class="branch-note"
          :class="note.tone"
          :style="{ left: `${note.x}px`, top: `${note.y}px`, width: `${note.width}px` }"
        >
          <strong>{{ note.title }}</strong>
          <ul>
            <li v-for="item in note.items" :key="item">{{ item }}</li>
          </ul>
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

        <div class="canvas-label label-start">Contexto</div>
        <div class="canvas-label label-core">Asignaciones de presupuesto</div>
        <div class="canvas-label label-output">Solicitud para liquidar</div>
      </div>
    </section>

    <nav v-if="!overviewMode" class="pager" aria-label="Navegacion del flujo">
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
    type: 'Contexto',
    title: 'Contexto del app',
    short: 'Solucion en Power Platform para solicitudes, presupuestos y liquidaciones.',
    text: 'ApproveCore centraliza solicitudes, asignaciones presupuestales y liquidaciones con flujos aprobatorios, trazabilidad y control financiero por proyecto.',
    detail: 'Esta rama explica el punto de partida: una aplicacion construida en Power Platform que conecta solicitudes, presupuesto, roles, flujos, tableros y control de estados.',
    tools: ['Control y trazabilidad de solicitudes.', 'Flujos de aprobacion jerarquicos.', 'Integracion con SAP, SharePoint y Power BI.', 'Reportes en tiempo real y seguridad por rol.'],
    example: 'Desde el contexto se abren dos ramas principales: asignaciones de presupuesto y solicitud para liquidar.',
    image: '/guide-screens/01_Portada_guia_visual.png',
    imageAlt: 'Pantalla de inicio y navegacion principal de ApproveCore',
    tone: 'info',
    x: 110,
    y: 1050
  },
  {
    id: 'budget',
    type: 'Rama 2',
    title: 'Asignaciones de presupuesto',
    short: 'Gestion de solicitudes para asignar presupuesto por proyecto y TDP.',
    text: 'La rama de presupuesto ordena roles, permisos y rutas de aprobacion para asignar TDP con control.',
    detail: 'Desde esta rama se explican los actores, permisos y flujos que procesan solicitudes de asignacion presupuestal.',
    tools: ['Solicitud de asignacion por proyecto y TDP.', 'Validaciones de monto, categoria y disponibilidad.', 'Asignadores CAPEX/OPEX y responsables por area.', 'Cierre trazable de la asignacion.'],
    example: 'El usuario entra a la rama de presupuesto y luego se distribuye por roles y tipo de flujo: PMO, OC/IEC, TA/ERC o SAQ.',
    tone: 'budget',
    x: 455,
    y: 580
  },
  {
    id: 'roles',
    type: '2.1',
    title: 'Roles y usuarios',
    short: 'Actores del proceso, permisos en el app y matriz RACI.',
    text: 'Define quienes participan, que permisos tienen y que responsabilidades asume cada rol.',
    detail: 'Esta tarjeta agrupa roles del negocio, permisos por perfil, asignacion de areas y matriz RACI del proceso.',
    tools: ['Roles: PMO, OC/IEC, TA/ERC, SAQ y liquidaciones.', 'Permisos por rol y responsabilidad.', 'Usuarios y asignacion de areas o portafolio.', 'Matriz RACI del proceso.'],
    example: 'Antes de abrir los flujos se confirma que cada usuario vea y ejecute solo las acciones correspondientes a su rol.',
    tone: 'roles',
    actionLeftOffset: 295,
    actionOffset: 48,
    x: 820,
    y: 580
  },
  {
    id: 'pmo-solicitante',
    type: '2.2 PMO',
    title: 'Solicitante PMO',
    short: 'Crea el ticket.',
    text: 'El solicitante PMO abre el ticket con datos del proyecto y sustento inicial.',
    detail: 'Primer paso del flujo PMO: registrar la solicitud para que pueda iniciar su aprobacion.',
    tools: ['Proyecto, etapa y actividad.', 'Monto, moneda y categoria.', 'Adjuntos y comentario de sustento.'],
    example: 'El ticket queda creado y pasa a revision del GP o gestor de portafolio.',
    tone: 'flow',
    variant: 'step',
    x: 1320,
    y: 60
  },
  {
    id: 'pmo-gp',
    type: '2.2 PMO',
    title: 'GP',
    short: 'Revision y aprobacion.',
    text: 'El gestor de portafolio revisa coherencia, alcance y prioridad.',
    detail: 'Segundo paso del flujo PMO: validar que la solicitud pertenezca al portafolio y tenga sustento suficiente.',
    tools: ['Revision de alcance.', 'Aprobacion o observacion.', 'Trazabilidad del responsable.'],
    example: 'Si la solicitud es correcta, avanza a gerencia.',
    tone: 'flow',
    variant: 'step',
    x: 1600,
    y: 60
  },
  {
    id: 'pmo-gerencia',
    type: '2.2 PMO',
    title: 'Gerencia',
    short: 'Revision y aprobacion.',
    text: 'Gerencia valida la solicitud antes de habilitar la asignacion presupuestal.',
    detail: 'Tercer paso PMO: aprobacion funcional y control de prioridad.',
    tools: ['Validacion gerencial.', 'Revision de presupuesto solicitado.', 'Aprobacion u observacion.'],
    example: 'La aprobacion gerencial habilita al asignador CAPEX/OPEX.',
    tone: 'flow',
    variant: 'step',
    x: 1880,
    y: 60
  },
  {
    id: 'pmo-asignador',
    type: '2.2 PMO',
    title: 'Asignador CAPEX / OPEX',
    short: 'Asignacion de TDP.',
    text: 'El asignador confirma la TDP y la informacion presupuestal asociada.',
    detail: 'Cuarto paso PMO: asignar o validar TDP, categoria y disponibilidad presupuestal.',
    tools: ['Asignacion de TDP.', 'Validacion de monto disponible.', 'Control CAPEX/OPEX.'],
    example: 'Con la TDP asignada, el ticket pasa a validacion final.',
    tone: 'flow',
    variant: 'step',
    x: 2160,
    y: 60
  },
  {
    id: 'pmo-asistente',
    type: '2.2 PMO',
    title: 'Asistente / liquidaciones',
    short: 'Validacion final.',
    text: 'El asistente confirma que la solicitud quede lista para cierre.',
    detail: 'Quinto paso PMO: revisar que la asignacion este completa y sin pendientes documentales.',
    tools: ['Validacion final.', 'Revision de evidencia.', 'Control de observaciones.'],
    example: 'Cuando todo esta conforme, el ticket pasa a cierre.',
    tone: 'flow',
    variant: 'step',
    x: 2440,
    y: 60
  },
  {
    id: 'pmo-cierre',
    type: '2.2 PMO',
    title: 'Cierre PMO',
    short: 'Asignacion completada.',
    text: 'El flujo PMO finaliza con la asignacion registrada y trazable.',
    detail: 'Ultimo paso PMO: cerrar el ticket con responsable, fecha y resultado.',
    tools: ['Estado final.', 'Responsable de cierre.', 'Historial disponible para consulta.'],
    example: 'La asignacion queda completada y disponible para seguimiento.',
    tone: 'flow',
    variant: 'step',
    x: 2720,
    y: 60
  },
  {
    id: 'ta-pmo-solicitante',
    type: '2.4 TA/ERC PMO',
    title: 'Solicitante',
    short: 'Genera ticket TA/ERC.',
    text: 'El solicitante con perfil PMO crea el ticket TA/ERC para iniciar la ruta especifica.',
    detail: 'Primer paso TA/ERC PMO: generar el ticket con sustento, datos del proyecto y requerimiento TA/ERC.',
    tools: ['Creacion de ticket TA/ERC.', 'Datos del proyecto y sustento.', 'Perfil PMO como origen del flujo.'],
    example: 'El ticket TA/ERC queda creado y pasa a revision de PM1, PM2 o PM3.',
    tone: 'taerc-pmo',
    variant: 'step',
    x: 1320,
    y: 330
  },
  {
    id: 'ta-pmo-pm',
    type: '2.4 TA/ERC PMO',
    title: 'PM 1 / PM 2 / PM 3',
    short: 'Revisar ticket.',
    text: 'El responsable PM revisa el ticket y confirma que procede.',
    detail: 'Segundo paso TA/ERC PMO: validacion del PM asignado segun portafolio o proyecto.',
    tools: ['Revision de ticket.', 'Validacion de proyecto.', 'Aprobacion u observacion.'],
    example: 'Con la revision PM aprobada, el ticket avanza a gerencia.',
    tone: 'taerc-pmo',
    variant: 'step',
    x: 1600,
    y: 330
  },
  {
    id: 'ta-pmo-gerencia',
    type: '2.4 TA/ERC PMO',
    title: 'Gerencia',
    short: 'Validar y aprobar.',
    text: 'Gerencia valida y aprueba el ticket TA/ERC.',
    detail: 'Tercer paso TA/ERC PMO: aprobacion gerencial antes de registrar TA o ERC.',
    tools: ['Validacion gerencial.', 'Aprobacion del ticket.', 'Registro de decision.'],
    example: 'La aprobacion habilita al asignador OPEX.',
    tone: 'taerc-pmo',
    variant: 'step',
    x: 1880,
    y: 330
  },
  {
    id: 'ta-pmo-asignador',
    type: '2.4 TA/ERC PMO',
    title: 'Asignador OPEX',
    short: 'Registrar TA/ERC.',
    text: 'El asignador OPEX registra la TA o ERC correspondiente.',
    detail: 'Cuarto paso TA/ERC PMO: registrar TA/ERC; este cierre no registra carrito.',
    tools: ['Registro de TA/ERC.', 'Control OPEX.', 'No se registra carrito en este flujo.'],
    example: 'Luego el ticket vuelve al solicitante para ingresar el codigo TA o ERC.',
    tone: 'taerc-pmo',
    variant: 'step',
    x: 2160,
    y: 330
  },
  {
    id: 'ta-pmo-ingreso',
    type: '2.4 TA/ERC PMO',
    title: 'Ingreso codigo TA/ERC',
    short: 'Solicitante ingresa codigo.',
    text: 'El solicitante ingresa el codigo TA o ERC para cerrar la ruta.',
    detail: 'Ultimo paso TA/ERC PMO: registrar el codigo final y dejar trazabilidad del cierre.',
    tools: ['Ingreso de codigo TA o ERC.', 'Cierre sin carrito.', 'Historial del ticket.'],
    example: 'El flujo finaliza con el codigo TA/ERC asociado al ticket.',
    tone: 'taerc-pmo',
    variant: 'step',
    x: 2440,
    y: 330
  },
  {
    id: 'oc-solicitante',
    type: '2.3 OC/IEC',
    title: 'Solicitante OC/IEC',
    short: 'Crea el ticket.',
    text: 'El solicitante OC/IEC registra la solicitud de operacion.',
    detail: 'Primer paso OC/IEC: crear el ticket con datos y sustento operativo.',
    tools: ['Datos del proyecto.', 'Categoria y presupuesto.', 'Adjuntos requeridos.'],
    example: 'El ticket pasa a subgerencia OC/IEC para revision.',
    tone: 'ociec',
    variant: 'step',
    x: 1320,
    y: 600
  },
  {
    id: 'oc-subgerente',
    type: '2.3 OC/IEC',
    title: 'Subgerente OC/IEC',
    short: 'Revision y aprobacion.',
    text: 'Subgerencia revisa la solicitud y valida su procedencia operativa.',
    detail: 'Segundo paso OC/IEC: validar alcance, necesidad y aprobacion del area.',
    tools: ['Revision operativa.', 'Aprobacion u observacion.', 'Registro del responsable.'],
    example: 'La aprobacion habilita la asignacion CAPEX/OPEX.',
    tone: 'ociec',
    variant: 'step',
    x: 1600,
    y: 600
  },
  {
    id: 'oc-asignador',
    type: '2.3 OC/IEC',
    title: 'Asignador CAPEX / OPEX',
    short: 'Asignacion de TDP.',
    text: 'El asignador registra la TDP y valida presupuesto.',
    detail: 'Tercer paso OC/IEC: asignacion presupuestal segun categoria y disponibilidad.',
    tools: ['Asignacion de TDP.', 'Validacion CAPEX/OPEX.', 'Control presupuestal.'],
    example: 'El ticket queda listo para validacion final del asistente.',
    tone: 'ociec',
    variant: 'step',
    x: 1880,
    y: 600
  },
  {
    id: 'oc-asistente',
    type: '2.3 OC/IEC',
    title: 'Asistente de gerencia',
    short: 'Validacion final.',
    text: 'El asistente de gerencia valida que la asignacion este completa.',
    detail: 'Cuarto paso OC/IEC: revisar evidencia, datos obligatorios y cierre pendiente.',
    tools: ['Validacion de evidencia.', 'Control de observaciones.', 'Confirmacion final.'],
    example: 'Si no hay pendientes, el ticket avanza a cierre.',
    tone: 'ociec',
    variant: 'step',
    x: 2160,
    y: 600
  },
  {
    id: 'oc-cierre',
    type: '2.3 OC/IEC',
    title: 'Cierre OC/IEC',
    short: 'Asignacion completada.',
    text: 'El flujo OC/IEC finaliza con asignacion registrada.',
    detail: 'Ultimo paso OC/IEC: cerrar la solicitud y dejar trazabilidad del resultado.',
    tools: ['Estado final.', 'Fecha y responsable.', 'Historial del ticket.'],
    example: 'La asignacion queda cerrada y lista para consulta posterior.',
    tone: 'ociec',
    variant: 'step',
    x: 2440,
    y: 600
  },
  {
    id: 'ta-solicitante',
    type: '2.4 TA/ERC',
    title: 'Solicitante',
    short: 'Crea ticket TA/ERC.',
    text: 'El solicitante abre la solicitud TA/ERC en proyectos de operacion.',
    detail: 'Primer paso TA/ERC: registrar una solicitud que requiere circuito operativo propio.',
    tools: ['Creacion de ticket TA/ERC.', 'Datos del proyecto.', 'Sustento inicial.'],
    example: 'El ticket pasa a Jefe OC para revision.',
    tone: 'taerc',
    variant: 'step',
    x: 1320,
    y: 870
  },
  {
    id: 'ta-jefe',
    type: '2.4 TA/ERC',
    title: 'Jefe OC',
    short: 'Revision y aprobacion.',
    text: 'Jefe OC valida la solicitud y su pertinencia operativa.',
    detail: 'Segundo paso TA/ERC: primera aprobacion funcional del circuito operativo.',
    tools: ['Revision del pedido.', 'Validacion del area.', 'Aprobacion u observacion.'],
    example: 'Si procede, avanza a subgerencia OC.',
    tone: 'taerc',
    variant: 'step',
    x: 1600,
    y: 870
  },
  {
    id: 'ta-subgerencia',
    type: '2.4 TA/ERC',
    title: 'Subgerencia OC',
    short: 'Revision y aprobacion.',
    text: 'Subgerencia OC valida el pedido antes de gerencia.',
    detail: 'Tercer paso TA/ERC: control intermedio para asegurar consistencia operativa.',
    tools: ['Revision de sustento.', 'Validacion de monto.', 'Aprobacion de subgerencia.'],
    example: 'Con la aprobacion, el ticket pasa a gerencia.',
    tone: 'taerc',
    variant: 'step',
    x: 1880,
    y: 870
  },
  {
    id: 'ta-gerencia',
    type: '2.4 TA/ERC',
    title: 'Gerencia',
    short: 'Revision y aprobacion.',
    text: 'Gerencia aprueba la continuidad del flujo TA/ERC.',
    detail: 'Cuarto paso TA/ERC: aprobacion gerencial del pedido operativo.',
    tools: ['Aprobacion gerencial.', 'Control de impacto.', 'Registro de decision.'],
    example: 'La aprobacion habilita al asignador OPEX.',
    tone: 'taerc',
    variant: 'step',
    x: 2160,
    y: 870
  },
  {
    id: 'ta-asignador',
    type: '2.4 TA/ERC',
    title: 'Asignador OPEX',
    short: 'Asignacion de TDP.',
    text: 'El asignador OPEX confirma la TDP de la solicitud.',
    detail: 'Quinto paso TA/ERC: asignacion presupuestal en la ruta OPEX.',
    tools: ['Asignacion de TDP.', 'Validacion de presupuesto.', 'Control OPEX.'],
    example: 'El flujo vuelve al solicitante para ingresar el codigo TA/ERC.',
    tone: 'taerc',
    variant: 'step',
    x: 2440,
    y: 870
  },
  {
    id: 'ta-ingreso',
    type: '2.4 TA/ERC',
    title: 'Ingreso TA/ERC',
    short: 'Solicitante ingresa TA/ERC.',
    text: 'El solicitante registra el codigo TA/ERC visible en su paso.',
    detail: 'Ultimo paso TA/ERC: ingresar el codigo y completar la transicion del flujo.',
    tools: ['Campo TA/ERC visible al solicitante.', 'Condiciones de visibilidad.', 'Cierre de transicion especifica.'],
    example: 'El codigo queda asociado al ticket para seguimiento posterior.',
    tone: 'taerc',
    variant: 'step',
    x: 2720,
    y: 870
  },
  {
    id: 'saq-solicitante',
    type: '2.5 SAQ',
    title: 'Solicitante SAQ',
    short: 'Crea el ticket.',
    text: 'El solicitante SAQ abre una solicitud de ruta simplificada.',
    detail: 'Primer paso SAQ: crear el ticket con validaciones basicas y ruta especifica.',
    tools: ['Creacion de ticket SAQ.', 'Ruta especifica 19.', 'Datos presupuestales minimos.'],
    example: 'El ticket pasa al asistente de gerencia SAQ.',
    tone: 'saq',
    variant: 'step',
    x: 1320,
    y: 1140
  },
  {
    id: 'saq-asistente',
    type: '2.5 SAQ',
    title: 'Asistente gerencia SAQ',
    short: 'Revision y aprobacion.',
    text: 'El asistente de gerencia SAQ revisa y aprueba la solicitud.',
    detail: 'Segundo paso SAQ: aprobacion directa y validacion presupuestal basica.',
    tools: ['Revision de datos.', 'Aprobacion directa.', 'Validaciones basicas de presupuesto.'],
    example: 'Si todo esta conforme, avanza al cierre del ticket.',
    tone: 'saq',
    variant: 'step',
    x: 1600,
    y: 1140
  },
  {
    id: 'saq-cierre',
    type: '2.5 SAQ',
    title: 'Cierre SAQ',
    short: 'Asignacion completada.',
    text: 'El flujo SAQ finaliza con la asignacion completada.',
    detail: 'Ultimo paso SAQ: cerrar la solicitud simplificada conservando trazabilidad.',
    tools: ['Cierre del ticket.', 'Resultado final.', 'Historial de aprobacion.'],
    example: 'La ruta SAQ queda completada sin pasos adicionales.',
    tone: 'saq',
    variant: 'step',
    x: 1880,
    y: 1140
  },
  {
    id: 'liquidation',
    type: 'Rama 3',
    title: 'Solicitud para liquidar',
    short: 'Gestion de TA/ERC, actas y documentos de liquidacion.',
    text: 'La rama de liquidacion registra solicitudes y valida documentacion hasta cerrar el estado del ticket.',
    detail: 'Esta rama explica como el solicitante abre la solicitud, como el liquidador revisa y como se controlan los estados.',
    tools: ['Formulario de creacion de ticket.', 'Datos del proyecto, TA/ERC, montos y adjuntos.', 'Revision documental y validaciones financieras.', 'Estados visibles para seguimiento y cierre.'],
    example: 'El usuario genera una solicitud para liquidar y el ticket avanza por apertura, revision, aprobacion, liquidacion y cierre.',
    tone: 'liquidation',
    x: 455,
    y: 1520
  },
  {
    id: 'opening',
    type: '3.1',
    title: 'Apertura de solicitud',
    short: 'El solicitante registra la solicitud para liquidar.',
    text: 'La pantalla de registro captura proyecto, TA/ERC, montos, adjuntos y datos obligatorios.',
    detail: 'El formulario debe explicar que se registra en cada campo para que el ticket nazca claro, trazable y listo para revision.',
    tools: ['Formulario de creacion de ticket.', 'Proyecto, TA/ERC, monto y adjuntos.', 'Validaciones de obligatoriedad.', 'Estados iniciales del ticket.'],
    example: 'La imagen muestra un ticket con campos llenados: proyecto, etapa, tipo de solicitud, categoria, moneda, monto, concepto, cantidad, TDP, adjuntos y comentario de sustento.',
    image: '/guide-screens/04_Generar_nuevos_tickets.png',
    imageAlt: 'Pantalla de generar ticket con datos principales llenados',
    tone: 'liquidation',
    x: 795,
    y: 1535
  },
  {
    id: 'liquidator',
    type: '3.2',
    title: 'Asignador / liquidador',
    short: 'Revision documental, validaciones y correcciones.',
    text: 'El asignador o liquidador revisa documentacion, valida finanzas y procesa la solicitud.',
    detail: 'El rol liquidador se enfoca en revisar sustento, corregir observaciones y conectar el proceso con SAP o control historico.',
    tools: ['Revision de documentacion.', 'Validaciones financieras.', 'Asignacion de liquidacion.', 'Observaciones y correcciones si aplica.', 'Integracion con SAP o control historico.'],
    example: 'El dashboard de carritos ayuda a revisar tickets con carrito pendiente, registros incompletos, estados financieros y responsables actuales.',
    image: '/guide-screens/05_Dashboard_carritos.png',
    imageAlt: 'Dashboard real de carritos y consulta operativa',
    tone: 'liquidation',
    x: 1135,
    y: 1535
  },
  {
    id: 'states',
    type: '3.3',
    title: 'Estados del ticket',
    short: 'Estados por los que puede pasar una solicitud de liquidacion.',
    text: 'Los estados explican donde esta el ticket y quien debe ejecutar la siguiente accion.',
    detail: 'El tablero de estados permite controlar trazabilidad, responsables, fechas y notificaciones por transicion.',
    tools: ['Pendiente: creado por el solicitante.', 'En revision: asignador o liquidador revisa.', 'Observado: con correcciones pendientes.', 'Aprobado: listo para liquidacion.', 'Liquidado: liquidacion completada.', 'Cerrado: ticket finalizado.'],
    example: 'El cierre confirma que no quedan acciones pendientes y que el ticket tiene estado final, responsable, fecha y evidencia suficiente.',
    states: ['Pendiente', 'En revision', 'Observado', 'Aprobado', 'Liquidado', 'Cerrado'],
    tone: 'states',
    width: 520,
    actionLeftOffset: 555,
    actionOffset: 36,
    x: 1505,
    y: 1505
  }
]

const paths = [
  {
    id: 'p1',
    kind: 'flow',
    to: 'budget',
    d: 'M380 1130 C455 1030 360 690 455 665'
  },
  {
    id: 'p2',
    kind: 'flow',
    to: 'liquidation',
    d: 'M380 1165 C455 1260 360 1570 455 1600'
  },
  {
    id: 'p3',
    kind: 'flow',
    to: 'roles',
    d: 'M725 665 C755 665 790 665 820 665'
  },
  {
    id: 'p4',
    kind: 'flow',
    to: 'pmo-solicitante',
    d: 'M1090 600 C1255 390 1140 125 1320 125'
  },
  {
    id: 'p5',
    kind: 'flow',
    to: 'ta-pmo-solicitante',
    d: 'M1090 630 C1235 525 1145 395 1320 395'
  },
  {
    id: 'p6',
    kind: 'flow',
    to: 'oc-solicitante',
    d: 'M1090 665 C1175 665 1235 665 1320 665'
  },
  {
    id: 'p7',
    kind: 'flow',
    to: 'ta-solicitante',
    d: 'M1090 700 C1235 805 1145 935 1320 935'
  },
  {
    id: 'p8',
    kind: 'flow',
    to: 'saq-solicitante',
    d: 'M1090 730 C1255 940 1140 1205 1320 1205'
  },
  {
    id: 'p9',
    kind: 'flow',
    to: 'pmo-gp',
    d: 'M1510 125 C1550 125 1560 125 1600 125'
  },
  {
    id: 'p10',
    kind: 'flow',
    to: 'pmo-gerencia',
    d: 'M1790 125 C1830 125 1840 125 1880 125'
  },
  {
    id: 'p11',
    kind: 'flow',
    to: 'pmo-asignador',
    d: 'M2070 125 C2110 125 2120 125 2160 125'
  },
  {
    id: 'p12',
    kind: 'flow',
    to: 'pmo-asistente',
    d: 'M2350 125 C2390 125 2400 125 2440 125'
  },
  {
    id: 'p13',
    kind: 'flow',
    to: 'pmo-cierre',
    d: 'M2630 125 C2670 125 2680 125 2720 125'
  },
  {
    id: 'p14',
    kind: 'flow',
    to: 'ta-pmo-pm',
    d: 'M1510 395 C1550 395 1560 395 1600 395'
  },
  {
    id: 'p15',
    kind: 'flow',
    to: 'ta-pmo-gerencia',
    d: 'M1790 395 C1830 395 1840 395 1880 395'
  },
  {
    id: 'p16',
    kind: 'flow',
    to: 'ta-pmo-asignador',
    d: 'M2070 395 C2110 395 2120 395 2160 395'
  },
  {
    id: 'p17',
    kind: 'flow',
    to: 'ta-pmo-ingreso',
    d: 'M2350 395 C2390 395 2400 395 2440 395'
  },
  {
    id: 'p18',
    kind: 'flow',
    to: 'oc-subgerente',
    d: 'M1510 665 C1550 665 1560 665 1600 665'
  },
  {
    id: 'p19',
    kind: 'flow',
    to: 'oc-asignador',
    d: 'M1790 665 C1830 665 1840 665 1880 665'
  },
  {
    id: 'p20',
    kind: 'flow',
    to: 'oc-asistente',
    d: 'M2070 665 C2110 665 2120 665 2160 665'
  },
  {
    id: 'p21',
    kind: 'flow',
    to: 'oc-cierre',
    d: 'M2350 665 C2390 665 2400 665 2440 665'
  },
  {
    id: 'p22',
    kind: 'flow',
    to: 'ta-jefe',
    d: 'M1510 935 C1550 935 1560 935 1600 935'
  },
  {
    id: 'p23',
    kind: 'flow',
    to: 'ta-subgerencia',
    d: 'M1790 935 C1830 935 1840 935 1880 935'
  },
  {
    id: 'p24',
    kind: 'flow',
    to: 'ta-gerencia',
    d: 'M2070 935 C2110 935 2120 935 2160 935'
  },
  {
    id: 'p25',
    kind: 'flow',
    to: 'ta-asignador',
    d: 'M2350 935 C2390 935 2400 935 2440 935'
  },
  {
    id: 'p26',
    kind: 'flow',
    to: 'ta-ingreso',
    d: 'M2630 935 C2670 935 2680 935 2720 935'
  },
  {
    id: 'p27',
    kind: 'flow',
    to: 'saq-asistente',
    d: 'M1510 1205 C1550 1205 1560 1205 1600 1205'
  },
  {
    id: 'p28',
    kind: 'flow',
    to: 'saq-cierre',
    d: 'M1790 1205 C1830 1205 1840 1205 1880 1205'
  },
  {
    id: 'p29',
    kind: 'generate',
    to: 'opening',
    d: 'M725 1600 C760 1600 760 1615 795 1615'
  },
  {
    id: 'p30',
    kind: 'generate',
    to: 'liquidator',
    d: 'M1065 1615 C1100 1615 1100 1615 1135 1615'
  },
  {
    id: 'p31',
    kind: 'generate',
    to: 'states',
    d: 'M1405 1615 C1450 1600 1455 1590 1505 1585'
  }
]

const notes = [
  {
    id: 'roles-detail',
    title: 'Detalles',
    tone: 'roles',
    x: 820,
    y: 785,
    width: 270,
    items: ['Jerarquias de aprobacion', 'Alcance por tipo de proyecto', 'Restricciones por rol en UI']
  },
  {
    id: 'pmo-detail',
    title: 'PMO',
    tone: 'flow',
    x: 2945,
    y: 72,
    width: 230,
    items: ['Implementacion', 'GP y gerencia', 'Asignacion CAPEX/OPEX']
  },
  {
    id: 'taerc-pmo-detail',
    title: 'TA/ERC PMO',
    tone: 'taerc-pmo',
    x: 2655,
    y: 342,
    width: 230,
    items: ['Solicitante genera ticket', 'PM 1 / PM 2 / PM 3 revisa', 'Asignador OPEX registra TA/ERC']
  },
  {
    id: 'oc-detail',
    title: 'OC/IEC',
    tone: 'ociec',
    x: 2655,
    y: 612,
    width: 230,
    items: ['Operacion', 'Subgerencia OC/IEC', 'Validacion final']
  },
  {
    id: 'taerc-detail',
    title: 'TA/ERC OC/IEC',
    tone: 'taerc',
    x: 2945,
    y: 882,
    width: 230,
    items: ['Codigo TA/ERC visible al solicitante', 'Condiciones de visibilidad', 'Estados y transiciones especificos']
  },
  {
    id: 'saq-detail',
    title: 'SAQ',
    tone: 'saq',
    x: 2095,
    y: 1152,
    width: 230,
    items: ['Ruta especifica 19', 'Aprobacion directa', 'Validaciones basicas de presupuesto']
  },
  {
    id: 'states-detail',
    title: 'Detalles',
    tone: 'states',
    x: 1505,
    y: 1768,
    width: 520,
    items: ['Trazabilidad completa de cambios y responsables', 'Fechas clave por estado', 'Notificaciones automaticas por transicion']
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
  { x: 720, y: 40, rotate: -0.8 },
  { x: 470, y: 230, rotate: 0.2 },
  { x: 250, y: 315, rotate: -0.4 },
  { x: -20, y: 340, rotate: 0.3 },
  { x: -300, y: 315, rotate: -0.2 },
  { x: -600, y: 295, rotate: 0.4 },
  { x: 430, y: -210, rotate: 0.4 },
  { x: 160, y: -230, rotate: -0.3 },
  { x: -120, y: -230, rotate: 0.3 },
  { x: -520, y: -190, rotate: -0.4 }
]

const canvasWidth = 3400
const canvasHeight = 2150
const canvasScale = 0.6
const overviewBounds = {
  minX: -80,
  minY: -40,
  maxX: 3420,
  maxY: 2080
}

const activeStep = ref(0)
const selectedNodeId = ref(null)
const focusedNodeId = ref(null)
const activePanel = ref(null)
const selectedRouteProfileId = ref(null)
const overviewMode = ref(false)
const viewportSize = ref({ width: 1280, height: 720 })

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
    left: `${selectedNode.value.x + (selectedNode.value.actionLeftOffset || 20)}px`,
    top: `${selectedNode.value.y + (selectedNode.value.actionOffset || (selectedNode.value.variant === 'step' ? 138 : 178))}px`
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
  if (overviewMode.value) {
    const margin = 180
    const overviewWidth = overviewBounds.maxX - overviewBounds.minX
    const overviewHeight = overviewBounds.maxY - overviewBounds.minY
    const overviewCenterX = overviewBounds.minX + overviewWidth / 2
    const overviewCenterY = overviewBounds.minY + overviewHeight / 2
    const fitScale = Math.min(
      (viewportSize.value.width - margin) / overviewWidth,
      (viewportSize.value.height - margin) / overviewHeight,
      0.6
    )
    const scale = Math.max(0.11, fitScale * 0.9)
    const x = Math.round((canvasWidth / 2 - overviewCenterX) * scale)
    const y = Math.round((canvasHeight / 2 - overviewCenterY) * scale)

    return {
      transform: `translate(-50%, -50%) translate3d(${x}px, ${y}px, 0) scale(${scale}) rotate(0deg)`
    }
  }

  const focusNode = nodes[cameraStep.value] || nodes[0]
  const focusWidth = focusNode.width || (focusNode.variant === 'step' ? 190 : 270)
  const focusHeight = focusNode.variant === 'step' ? 132 : 170
  const x = Math.round((canvasWidth / 2 - (focusNode.x + focusWidth / 2)) * canvasScale)
  const y = Math.round((canvasHeight / 2 - (focusNode.y + focusHeight / 2)) * canvasScale)
  const rotate = focusNode.variant === 'step' ? 0.15 : -0.4

  return {
    transform: `translate(-50%, -50%) translate3d(${x}px, ${y}px, 0) scale(${canvasScale}) rotate(${rotate}deg)`
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
  updateViewportSize()
  window.addEventListener('keydown', handleKeyboardNavigation, true)
  window.addEventListener('resize', updateViewportSize)
})

onBeforeUnmount(() => {
  window.removeEventListener('keydown', handleKeyboardNavigation, true)
  window.removeEventListener('resize', updateViewportSize)
})

const selectNode = (nodeId) => {
  const nodeIndex = nodes.findIndex((node) => node.id === nodeId)
  const isSelected = selectedNodeId.value === nodeId

  overviewMode.value = false
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

const toggleOverview = () => {
  overviewMode.value = !overviewMode.value
  activePanel.value = null
}

const updateViewportSize = () => {
  viewportSize.value = {
    width: window.innerWidth,
    height: window.innerHeight
  }
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
  gap: 7px;
  z-index: 12;
}

.progress span {
  width: 7px;
  height: 7px;
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
  width: 20px;
}

.progress.overview {
  opacity: 0;
  pointer-events: none;
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

.overview-toggle {
  position: fixed;
  left: clamp(112px, 10vw, 160px);
  top: clamp(24px, 4vh, 44px);
  z-index: 26;
  min-width: 92px;
  padding: 9px 12px;
  border: 1px solid #d7cabd;
  border-radius: 6px;
  color: #7c2d12;
  background: rgba(250, 247, 241, 0.92);
  box-shadow: 0 14px 30px rgba(92, 67, 49, 0.1);
  cursor: pointer;
  font: inherit;
  font-size: 11px;
  font-weight: 900;
  letter-spacing: 0.06em;
  line-height: 1;
  text-transform: uppercase;
}

.overview-toggle:hover,
.overview-toggle:focus-visible,
.overview-toggle[aria-pressed="true"] {
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
  width: 3400px;
  height: 2150px;
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
  stroke: rgba(31, 111, 149, 0.74);
  stroke-width: 2.4;
  stroke-linecap: round;
  stroke-dasharray: 11 20;
  opacity: 1;
  animation: dash-flow 820ms linear infinite;
  transition: stroke 450ms ease, stroke-width 450ms ease;
}

.flow-path.generate {
  stroke: rgba(31, 111, 149, 0.74);
  stroke-dasharray: 11 20;
  animation-duration: 820ms;
}

.flow-path.lit {
  stroke: #1f6f95;
  stroke-width: 3;
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
  min-height: 164px;
  padding: 16px 18px;
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
  gap: 12px;
  margin-bottom: 12px;
  color: #8d7b6d;
  font-size: 11px;
  font-weight: 800;
  letter-spacing: 0.14em;
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
  font-size: 24px;
  font-weight: 900;
  line-height: 1.02;
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

.node-summary {
  margin: 12px 0 0;
  color: #6f6258;
  font-size: 13px;
  font-weight: 800;
  line-height: 1.32;
}

.node.step {
  width: 190px;
  min-height: 132px;
  padding: 14px 14px;
}

.node.step .node-topline {
  margin-bottom: 10px;
  font-size: 9px;
  letter-spacing: 0.08em;
}

.node.step .node-index {
  width: 24px;
  height: 24px;
  font-size: 11px;
}

.node.step .node-title {
  min-height: 0;
  font-size: 18px;
  line-height: 1.08;
}

.node.step .node-summary {
  margin-top: 9px;
  font-size: 11px;
  line-height: 1.28;
}

.node-steps {
  display: grid;
  gap: 7px;
  margin: 14px 0 0;
  padding: 0;
  list-style: none;
}

.node-steps li {
  display: grid;
  grid-template-columns: 28px 1fr;
  align-items: center;
  gap: 9px;
  min-height: 34px;
  padding: 6px 8px;
  border: 1px solid rgba(31, 111, 149, 0.18);
  border-radius: 5px;
  background: rgba(255, 250, 242, 0.72);
}

.node-steps span {
  display: grid;
  place-items: center;
  width: 24px;
  height: 24px;
  border-radius: 999px;
  color: #fff8ef;
  background: #1f6f95;
  font-size: 13px;
  font-weight: 900;
}

.node-steps strong {
  color: #4f3e32;
  font-size: 12px;
  font-weight: 900;
  line-height: 1.1;
}

.state-strip {
  display: grid;
  grid-template-columns: repeat(6, minmax(0, 1fr));
  gap: 9px;
  margin-top: 16px;
}

.state-strip span {
  display: grid;
  place-items: center;
  min-height: 54px;
  padding: 8px 6px;
  border: 1px solid rgba(31, 111, 149, 0.24);
  border-radius: 5px;
  color: #4f3e32;
  background: rgba(255, 250, 242, 0.74);
  font-size: 10px;
  font-weight: 900;
  line-height: 1.1;
  text-align: center;
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

.node.info {
  border-color: rgba(31, 111, 149, 0.38);
  background: rgba(221, 232, 236, 0.86);
}

.node.budget {
  border-color: rgba(34, 112, 76, 0.42);
  background: rgba(227, 241, 225, 0.9);
}

.node.roles {
  border-color: rgba(93, 132, 70, 0.4);
  background: rgba(242, 247, 233, 0.92);
}

.node.flow {
  border-color: rgba(31, 111, 149, 0.38);
  background: rgba(233, 241, 247, 0.92);
}

.node.ociec {
  border-color: rgba(99, 95, 162, 0.36);
  background: rgba(239, 240, 250, 0.92);
}

.node.taerc {
  border-color: rgba(177, 111, 15, 0.42);
  background: rgba(252, 242, 220, 0.94);
}

.node.taerc-pmo {
  border-color: rgba(150, 117, 23, 0.42);
  background: rgba(250, 244, 219, 0.94);
}

.node.saq {
  border-color: rgba(198, 88, 24, 0.42);
  background: rgba(252, 235, 222, 0.94);
}

.node.liquidation,
.node.states {
  border-color: rgba(112, 71, 155, 0.36);
  background: rgba(242, 237, 248, 0.92);
}

.node.budget .node-index,
.node.roles .node-index {
  background: #32764e;
}

.node.taerc .node-index {
  background: #b16f0f;
}

.node.taerc-pmo .node-index {
  background: #967517;
}

.node.saq .node-index {
  background: #c65818;
}

.node.liquidation .node-index,
.node.states .node-index {
  background: #70479b;
}

.node.ociec .node-index {
  background: #5f62a8;
}

.node.budget .node-title,
.node.roles .node-title {
  color: #2f6d49;
}

.node.taerc .node-title {
  color: #935d0d;
}

.node.taerc-pmo .node-title {
  color: #7c6215;
}

.node.saq .node-title {
  color: #a54a17;
}

.node.liquidation .node-title,
.node.states .node-title {
  color: #5f3d82;
}

.node.ociec .node-title {
  color: #4e5296;
}

.branch-note {
  position: absolute;
  z-index: 2;
  padding: 14px 16px;
  border: 1px dashed rgba(124, 45, 18, 0.24);
  border-radius: 5px;
  color: #4f3e32;
  background: rgba(250, 247, 241, 0.78);
  box-shadow: 0 10px 22px rgba(92, 67, 49, 0.06);
}

.branch-note strong {
  display: block;
  margin-bottom: 8px;
  color: #7c2d12;
  font-size: 13px;
  font-weight: 900;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

.branch-note ul {
  display: grid;
  gap: 6px;
  margin: 0;
  padding: 0 0 0 15px;
}

.branch-note li {
  color: #4f3e32;
  font-size: 12px;
  font-weight: 800;
  line-height: 1.25;
}

.branch-note.roles {
  border-color: rgba(93, 132, 70, 0.34);
  background: rgba(242, 247, 233, 0.82);
}

.branch-note.flow {
  border-color: rgba(31, 111, 149, 0.32);
  background: rgba(233, 241, 247, 0.82);
}

.branch-note.ociec {
  border-color: rgba(99, 95, 162, 0.32);
  background: rgba(239, 240, 250, 0.82);
}

.branch-note.taerc {
  border-color: rgba(177, 111, 15, 0.34);
  background: rgba(252, 242, 220, 0.84);
}

.branch-note.taerc-pmo {
  border-color: rgba(150, 117, 23, 0.34);
  background: rgba(250, 244, 219, 0.84);
}

.branch-note.saq {
  border-color: rgba(198, 88, 24, 0.34);
  background: rgba(252, 235, 222, 0.84);
}

.branch-note.states {
  border-color: rgba(112, 71, 155, 0.32);
  background: rgba(242, 237, 248, 0.82);
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
  left: 2800px;
  top: 1450px;
}

.cluster-automation {
  left: 2810px;
  top: 1690px;
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
  left: 110px;
  top: 985px;
}

.label-core {
  left: 455px;
  top: 520px;
}

.label-output {
  left: 795px;
  top: 1475px;
}

.pager {
  position: fixed;
  right: clamp(18px, 3vw, 36px);
  bottom: clamp(22px, 4vh, 42px);
  z-index: 24;
  display: grid;
  grid-template-columns: 30px minmax(118px, 178px) 30px;
  align-items: center;
  gap: 8px;
  min-height: 48px;
  padding: 8px 10px;
  border: 1px solid #d7cabd;
  border-radius: 6px;
  background: rgba(250, 247, 241, 0.92);
  box-shadow: 0 18px 44px rgba(92, 67, 49, 0.12);
}

.pager-btn {
  display: grid;
  place-items: center;
  width: 28px;
  height: 28px;
  border: 2px solid #d7cabd;
  border-radius: 5px;
  color: #7f766d;
  background: rgba(255, 250, 242, 0.96);
  cursor: pointer;
  font: inherit;
  font-size: 21px;
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
  font-size: 12px;
  font-weight: 900;
  line-height: 1;
  letter-spacing: 0.08em;
}

.pager-current strong {
  color: #7c2d12;
  font-size: 14px;
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
    grid-template-columns: 30px minmax(102px, 158px) 30px;
    gap: 7px;
    width: min(82vw, 250px);
    min-height: 48px;
    padding: 8px 9px;
  }

  .pager-btn {
    width: 27px;
    height: 27px;
    font-size: 20px;
  }

  .pager-current span {
    font-size: 11px;
  }

  .pager-current strong {
    font-size: 13px;
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
