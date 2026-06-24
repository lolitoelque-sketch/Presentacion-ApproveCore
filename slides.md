---
layout: full
---

<section class="title-cover">
  <div class="cover-eyebrow">APPRUVECORE</div>
  <h1>Gestion y trazabilidad de solicitudes presupuestales</h1>
  <p>
    Una presentacion para explicar el paso de un proceso basado en correos a un flujo
    controlado, medible y visible de inicio a fin.
  </p>
  <div class="cover-flow">
    <span>Solicitud</span>
    <i />
    <span>Aprobacion</span>
    <i />
    <span>Asignacion</span>
    <i />
    <span>Carrito / PO</span>
    <i />
    <span>Cierre</span>
  </div>
</section>

<style>
:root {
  --ap-blue: #8bd7ff;
  --ap-orange: #ffb28a;
  --ap-ink: #0b0f14;
  --ap-panel: rgba(11, 15, 20, 0.76);
  --ap-muted: #8b94a6;
  --ap-text: #d7deea;
}

.title-cover {
  position: fixed;
  inset: 0;
  display: grid;
  align-content: start;
  justify-items: start;
  gap: 28px;
  padding: clamp(70px, 10vh, 110px) clamp(32px, 5vw, 70px);
  color: var(--ap-blue);
  background:
    radial-gradient(circle at 28% 22%, rgba(125, 211, 252, 0.08), transparent 28%),
    radial-gradient(circle at 78% 76%, rgba(255, 178, 138, 0.08), transparent 30%),
    linear-gradient(135deg, #0b0f14 0%, #10141b 52%, #0b0f14 100%);
  overflow: hidden;
  font-family: "Cascadia Code", "JetBrains Mono", "Fira Code", Consolas, ui-monospace, monospace;
}

.title-cover::before {
  content: "";
  position: absolute;
  inset: -80px;
  background-image:
    radial-gradient(circle, rgba(139, 148, 166, 0.22) 1px, transparent 1.5px),
    linear-gradient(rgba(125, 211, 252, 0.06) 1px, transparent 1px),
    linear-gradient(90deg, rgba(125, 211, 252, 0.06) 1px, transparent 1px);
  background-size: 40px 40px, 160px 160px, 160px 160px;
  transform: rotate(-2deg);
}

.cover-eyebrow,
.title-cover h1,
.title-cover p,
.cover-flow {
  position: relative;
}

.cover-eyebrow {
  color: var(--ap-muted);
  font-size: 16px;
  font-weight: 900;
  letter-spacing: 0.32em;
}

.title-cover h1 {
  max-width: 920px;
  margin: 0;
  font-size: clamp(34px, 4vw, 58px);
  line-height: 1.05;
  letter-spacing: 0;
  text-align: left;
  font-weight: 900;
  text-shadow: 0 0 36px rgba(125, 211, 252, 0.16);
}

.title-cover p {
  max-width: 850px;
  margin: 0;
  color: #d7deea;
  font-size: clamp(18px, 1.8vw, 25px);
  font-weight: 700;
  line-height: 1.45;
}

.cover-flow {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 12px;
  max-width: 980px;
  margin-top: 10px;
}

.cover-flow span {
  padding: 11px 14px;
  border: 1px solid rgba(139, 215, 255, 0.34);
  border-radius: 4px;
  color: #d7deea;
  background: rgba(11, 15, 20, 0.7);
  font-size: 15px;
  font-weight: 900;
  text-transform: uppercase;
}

.cover-flow i {
  width: 34px;
  height: 2px;
  background: var(--ap-orange);
}

.ap-slide {
  position: fixed;
  inset: 0;
  display: grid;
  align-content: center;
  gap: 32px;
  padding: clamp(52px, 7vh, 82px) clamp(34px, 6vw, 86px);
  color: var(--ap-text);
  background:
    radial-gradient(circle at 18% 16%, rgba(125, 211, 252, 0.08), transparent 30%),
    radial-gradient(circle at 82% 78%, rgba(255, 178, 138, 0.07), transparent 28%),
    linear-gradient(135deg, #0b0f14 0%, #10141b 48%, #0b0f14 100%);
  overflow: hidden;
  font-family: "Cascadia Code", "JetBrains Mono", "Fira Code", Consolas, ui-monospace, monospace;
}

.ap-slide::before {
  content: "";
  position: absolute;
  inset: -80px;
  background-image:
    linear-gradient(rgba(125, 211, 252, 0.055) 1px, transparent 1px),
    linear-gradient(90deg, rgba(125, 211, 252, 0.055) 1px, transparent 1px);
  background-size: 52px 52px;
  transform: rotate(-2deg) scale(1.05);
}

.ap-slide > * {
  position: relative;
}

.ap-kicker {
  color: var(--ap-muted);
  font-size: 14px;
  font-weight: 900;
  letter-spacing: 0.22em;
  text-transform: uppercase;
}

.ap-slide h2 {
  max-width: 1040px;
  margin: 0;
  color: var(--ap-blue);
  font-size: clamp(32px, 4.2vw, 58px);
  font-weight: 900;
  line-height: 1.05;
  letter-spacing: 0;
}

.ap-slide p {
  max-width: 1040px;
  margin: 0;
  color: #f8fafc;
  font-size: clamp(19px, 1.9vw, 26px);
  font-weight: 700;
  line-height: 1.45;
}

.ap-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 18px;
}

.ap-card,
.ap-step,
.ap-role,
.ap-metric {
  min-height: 150px;
  padding: 22px;
  border: 1px solid rgba(139, 148, 166, 0.32);
  border-radius: 4px;
  background: var(--ap-panel);
  backdrop-filter: blur(12px);
}

.ap-card strong,
.ap-step strong,
.ap-role strong,
.ap-metric strong {
  display: block;
  margin-bottom: 12px;
  color: var(--ap-orange);
  font-size: 20px;
  font-weight: 900;
  line-height: 1.15;
}

.ap-card span,
.ap-step span,
.ap-role span,
.ap-metric span {
  color: #d7deea;
  font-size: 16px;
  font-weight: 700;
  line-height: 1.35;
}

.ap-two {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 22px;
}

.ap-list {
  display: grid;
  gap: 14px;
  margin: 0;
  padding: 0;
  list-style: none;
}

.ap-list li {
  padding: 16px 18px;
  border-left: 4px solid var(--ap-orange);
  color: #e5e7eb;
  background: rgba(11, 15, 20, 0.64);
  font-size: 19px;
  font-weight: 800;
  line-height: 1.3;
}

.ap-flowline {
  display: grid;
  grid-template-columns: repeat(5, minmax(0, 1fr));
  gap: 14px;
}

.ap-step {
  min-height: 170px;
}

.ap-step small {
  display: block;
  margin-bottom: 12px;
  color: var(--ap-blue);
  font-size: 13px;
  font-weight: 900;
  letter-spacing: 0.18em;
  text-transform: uppercase;
}

.ap-wide {
  grid-column: span 2;
}

@media (max-width: 900px) {
  .ap-grid,
  .ap-two,
  .ap-flowline {
    grid-template-columns: 1fr;
  }
}
</style>

---
layout: full
---

<section class="ap-slide">
  <div class="ap-kicker">Contexto</div>
  <h2>El proceso necesitaba pasar de coordinaciones manuales a tickets trazables.</h2>
  <p>
    Las solicitudes se gestionaban con correos, archivos adjuntos y seguimiento individual.
    Eso hacia dificil saber el estado real, el responsable actual y la informacion pendiente
    para avanzar.
  </p>
  <div class="ap-grid">
    <div class="ap-card"><strong>Visibilidad limitada</strong><span>No existia una bandeja unica para consultar el avance de cada solicitud.</span></div>
    <div class="ap-card"><strong>Responsables dispersos</strong><span>La siguiente accion dependia de cadenas de correo y coordinaciones fuera del sistema.</span></div>
    <div class="ap-card"><strong>Control financiero fragil</strong><span>Montos, carritos, PO y estados podian quedar repartidos en varias fuentes.</span></div>
  </div>
</section>

---
layout: full
---

<section class="ap-slide">
  <div class="ap-kicker">Solucion</div>
  <h2>AppruveCore centraliza solicitudes, aprobaciones, asignaciones y cierre operativo.</h2>
  <p>
    La aplicacion convierte cada necesidad en un ticket con datos estructurados, ruta de
    aprobacion, responsable visible, asignaciones financieras y evidencia para auditoria.
  </p>
  <div class="ap-grid">
    <div class="ap-card"><strong>Tickets</strong><span>Registro de proyecto, actividad, subactividad, monto, moneda, TDP y sustentos.</span></div>
    <div class="ap-card"><strong>Flujos</strong><span>Aprobaciones por tipo de solicitud, area responsable y rol configurado.</span></div>
    <div class="ap-card"><strong>Seguimiento</strong><span>Carrito, TA/ERC, PO, estados financieros, observaciones y cierre.</span></div>
  </div>
</section>

---
layout: full
---

<section class="ap-slide">
  <div class="ap-kicker">Beneficios</div>
  <h2>La app permite saber que se solicito, quien lo aprobo, donde esta detenido y que falta.</h2>
  <div class="ap-two">
    <ul class="ap-list">
      <li>Centralizacion de solicitudes y bandejas de control.</li>
      <li>Trazabilidad completa del ticket y sus cambios.</li>
      <li>Menor dependencia del correo y menos reprocesos.</li>
      <li>Responsables visibles por estado y flujo.</li>
    </ul>
    <ul class="ap-list">
      <li>Validacion por roles definidos.</li>
      <li>Seguimiento de carritos, PO y estados financieros.</li>
      <li>Control presupuestal por proyecto, etapa y TDP.</li>
      <li>Informacion historica disponible para consulta y auditoria.</li>
    </ul>
  </div>
</section>

---
layout: full
---

<section class="ap-slide">
  <div class="ap-kicker">Herramientas</div>
  <h2>La solucion aprovecha el ecosistema Microsoft disponible en la organizacion.</h2>
  <div class="ap-grid">
    <div class="ap-card"><strong>Power Apps</strong><span>Interfaz principal para crear tickets, aprobar, observar, consultar y cerrar.</span></div>
    <div class="ap-card"><strong>SharePoint Lists</strong><span>Base estructurada para tickets, asignaciones, usuarios, proyectos, rutas y catalogos.</span></div>
    <div class="ap-card"><strong>Power Automate</strong><span>Automatizacion de actualizaciones, sincronizacion SAP y procesos complementarios.</span></div>
    <div class="ap-card"><strong>Power BI</strong><span>Analisis presupuestal y visualizacion de avance por proyecto.</span></div>
    <div class="ap-card"><strong>Microsoft 365</strong><span>Integracion con usuarios corporativos, permisos y operacion diaria.</span></div>
    <div class="ap-card"><strong>SAP / PO</strong><span>Conexion operativa con correlativos, carritos, PO y seguimiento financiero.</span></div>
  </div>
</section>

---
layout: full
---

<InfiniteFlow />

---
layout: full
---

<section class="ap-slide">
  <div class="ap-kicker">Arquitectura funcional</div>
  <h2>Cada accion del usuario queda registrada en datos estructurados.</h2>
  <div class="ap-flowline">
    <div class="ap-step"><small>01</small><strong>Power Apps</strong><span>Entrada del usuario y pantallas por rol.</span></div>
    <div class="ap-step"><small>02</small><strong>SharePoint</strong><span>Tickets, asignaciones, usuarios, rutas, proyectos y catalogos.</span></div>
    <div class="ap-step"><small>03</small><strong>Power Automate</strong><span>Actualizaciones, sincronizaciones y procesos de soporte.</span></div>
    <div class="ap-step"><small>04</small><strong>SAP / PO</strong><span>Correlativos, carritos, TA/ERC y avance financiero.</span></div>
    <div class="ap-step"><small>05</small><strong>Dashboards</strong><span>Lectura consolidada del presupuesto y estados.</span></div>
  </div>
</section>

---
layout: full
---

<section class="ap-slide">
  <div class="ap-kicker">Pantallas principales</div>
  <h2>La aplicacion se organiza alrededor de las etapas naturales del ticket.</h2>
  <div class="ap-grid">
    <div class="ap-card"><strong>Menu principal</strong><span>Entrada central con opciones segun perfil.</span></div>
    <div class="ap-card"><strong>Creacion de tickets</strong><span>Registro de proyecto, monto, TDP, asignaciones y sustentos.</span></div>
    <div class="ap-card"><strong>Historial</strong><span>Consulta de estados, fechas, responsables y detalle del flujo.</span></div>
    <div class="ap-card"><strong>Bandejas</strong><span>Aprobaciones, observaciones y acciones pendientes por rol.</span></div>
    <div class="ap-card"><strong>Carritos</strong><span>Control de carrito, TA/ERC, correlativo SAP y PO.</span></div>
    <div class="ap-card"><strong>Presupuesto</strong><span>Dashboard, tarjetero y estado financiero de asignaciones.</span></div>
  </div>
</section>

---
layout: full
---

<section class="ap-slide">
  <div class="ap-kicker">Proceso E2E</div>
  <h2>El ticket tiene una vida completa y trazable dentro de la aplicacion.</h2>
  <div class="ap-flowline">
    <div class="ap-step"><small>Paso 1</small><strong>Nacimiento</strong><span>El solicitante registra la necesidad, proyecto, monto, TDP y sustentos.</span></div>
    <div class="ap-step"><small>Paso 2</small><strong>Aprobacion</strong><span>La app dirige el ticket al responsable segun ruta y flujo.</span></div>
    <div class="ap-step"><small>Paso 3</small><strong>Validacion</strong><span>Los aprobadores revisan datos, adjuntos, coherencia y presupuesto.</span></div>
    <div class="ap-step"><small>Paso 4</small><strong>Asignacion</strong><span>Se valida CAPEX/OPEX, TDP, centro de costo y correlativo SAP.</span></div>
    <div class="ap-step"><small>Paso 5</small><strong>Cierre</strong><span>Se registra carrito, TA/ERC o PO y queda historial para auditoria.</span></div>
  </div>
</section>

---
layout: full
---

<section class="ap-slide">
  <div class="ap-kicker">Flujos principales</div>
  <h2>No todas las solicitudes siguen el mismo camino, pero todas mantienen la misma trazabilidad.</h2>
  <div class="ap-grid">
    <div class="ap-card"><strong>PMO</strong><span>Solicitante PMO, PM o GP, gerencia, asignadores CAPEX/OPEX y cierre operativo.</span></div>
    <div class="ap-card"><strong>OC / IEC</strong><span>Validacion jerarquica operativa antes de pasar a asignacion y seguimiento final.</span></div>
    <div class="ap-card"><strong>TA / ERC</strong><span>El solicitante vuelve al final para registrar el codigo TA/ERC cuando corresponde.</span></div>
    <div class="ap-card"><strong>Reembolsos</strong><span>Sustento, validacion administrativa o presupuestal y cierre por responsable final.</span></div>
    <div class="ap-card ap-wide"><strong>Logica comun</strong><span>Registrar, aprobar, observar si hace falta, asignar presupuesto, ejecutar y cerrar con evidencia.</span></div>
  </div>
</section>

---
layout: full
---

<section class="ap-slide">
  <div class="ap-kicker">Usuarios y aprobadores</div>
  <h2>Cada usuario ve y ejecuta acciones segun su responsabilidad dentro del proceso.</h2>
  <div class="ap-grid">
    <div class="ap-role"><strong>Solicitante</strong><span>Crea tickets, registra asignaciones, adjunta sustentos, corrige observaciones y hace seguimiento.</span></div>
    <div class="ap-role"><strong>Aprobadores</strong><span>Validan informacion, montos, sustentos y coherencia con el proyecto.</span></div>
    <div class="ap-role"><strong>Asignadores CAPEX/OPEX</strong><span>Revisan TDP, centro de costo, presupuesto y consistencia financiera.</span></div>
    <div class="ap-role"><strong>Asistente / Liquidaciones</strong><span>Registran carrito, validan avance operativo, apoyan PO y cierre.</span></div>
    <div class="ap-role"><strong>Controller</strong><span>Consulta presupuesto, estados financieros e informacion consolidada.</span></div>
    <div class="ap-role"><strong>Administrador funcional</strong><span>Mantiene usuarios, rutas, catalogos y reglas de visibilidad.</span></div>
  </div>
</section>

---
layout: full
---

<section class="ap-slide">
  <div class="ap-kicker">Demo sugerida</div>
  <h2>La capacitacion debe mostrar lo que cada usuario hara en su dia a dia.</h2>
  <div class="ap-two">
    <ul class="ap-list">
      <li>Ingresar al menu principal y crear un ticket.</li>
      <li>Revisar asignaciones, historial y responsable actual.</li>
      <li>Aprobar un ticket como aprobador.</li>
      <li>Observar, corregir y retornar al flujo normal.</li>
    </ul>
    <ul class="ap-list">
      <li>Asignar TDP o presupuesto.</li>
      <li>Registrar carrito o codigo TA/ERC.</li>
      <li>Consultar dashboard, tarjetero y estado financiero.</li>
      <li>Cerrar con el resumen completo del ticket.</li>
    </ul>
  </div>
</section>

---
layout: full
---

<section class="ap-slide">
  <div class="ap-kicker">Cierre</div>
  <h2>AppruveCore es una base tecnologica para estandarizar y escalar el control de solicitudes.</h2>
  <div class="ap-grid">
    <div class="ap-metric"><strong>Menos correos</strong><span>La coordinacion se convierte en flujo controlado.</span></div>
    <div class="ap-metric"><strong>Mas trazabilidad</strong><span>Estados, responsables, observaciones y decisiones quedan registrados.</span></div>
    <div class="ap-metric"><strong>Mayor control</strong><span>Presupuesto, carrito, PO y cierre se consultan desde la aplicacion.</span></div>
    <div class="ap-metric"><strong>Proximos pasos</strong><span>Marcha blanca, capacitacion por perfil, feedback y ampliacion progresiva.</span></div>
  </div>
</section>
