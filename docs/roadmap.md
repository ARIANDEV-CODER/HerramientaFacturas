## Roadmap de implementación

### Principios
- Iteraciones cortas con entregables usables desde las primeras semanas.
- Código 100 % testeado (unitario + feature + e2e front).
- Infraestructura replicable (scripts/artisan + docker).
- Documentación viva (README, docs técnicos, guías para el cliente).

---

### Fase 0 · Descubrimiento (pendiente de facturas de referencia)
1. Analizar las dos facturas y detectar:
   - Campos obligatorios y opcionales.
   - Secciones visuales, tipografías y estilos.
   - Textos legales, firmas y condiciones especiales.
2. Validar con el cliente:
   - Hosting final (cPanel vs VPS/Docker).
   - Idiomas/monedas iniciales.
   - Necesidad de multiempresa/multiusuario desde el MVP.
3. Cerrar backlog priorizado y definir KPIs de éxito.

**Duración estimada**: 2-3 días tras recibir las facturas.

---

### Fase 1 · Fundaciones técnicas
1. Inicializar monorepo (Laravel backend + React frontend) con tooling compartido.
2. Configurar CI (GitHub Actions) para tests, linters, builds y análisis estático.
3. Preparar scripts de desarrollo (`make`, `npm`, `artisan`) y entorno Docker opcional.
4. Definir convenciones de código, commits y ramas.

**Duración estimada**: 3 días.

---

### Fase 2 · Backend Core (Laravel)
1. Migraciones + seeders para `companies`, `clients`, `documents`, `document_items`, `taxes`, `settings`, `media`, `users`.
2. Servicios de dominio:
   - Numeración automática.
   - Cálculo de impuestos y totales.
   - Conversión de presupuesto → factura.
3. API REST (Sanctum) + Policy/RBAC.
4. Uploads de logos y storage S3/local.
5. Test unitarios y feature.

**Duración estimada**: 1.5 semanas.

---

### Fase 3 · Frontend SPA (React + Tailwind)
1. Setup base (Routing, Zustand/Redux, React Query, diseño responsivo).
2. Componentes globales (inputs, tablas, cards, modales, toasts).
3. Flujo principal:
   - Landing con CTA.
   - Wizard de creación (empresa → cliente → cabecera → líneas → logo → review).
   - Gestión de clientes y documentos ya creados.
4. Previsualización en vivo del template.
5. Tests unitarios + e2e (Playwright/Cypress).

**Duración estimada**: 2 semanas.

---

### Fase 4 · PDFs, envíos y automatizaciones
1. Plantillas Blade + estilos premium (factura y presupuesto).
2. Render backend (Snappy/Browsershot) + almacenamiento y versionado.
3. Emails transaccionales (sendgrid/mailgun) con adjuntos.
4. Scheduler para recordatorios de vencimiento y limpieza de borradores.
5. Métricas y logging (Telescope, Monolog, alertas).

**Duración estimada**: 1 semana.

---

### Fase 5 · Hardening y despliegue
1. QA exhaustivo (checklists funcionales, pruebas de carga sobre endpoints críticos).
2. Ajustes de seguridad (headers, rate limiting, backups).
3. Documentación final (manual de usuario, guía de despliegue, runbooks).
4. Puesta en producción:
   - Hosting compartido: build + migraciones + seeds + jobs.
   - VPS/Docker: docker-compose/k8s + certificados TLS + monitoreo.
5. Handoff + plan de soporte.

**Duración estimada**: 1 semana.

---

### Cronograma ejemplo

| Semana | Entregable |
|--------|------------|
| 0 | Facturas analizadas + backlog final |
| 1 | Monorepo y CI listos |
| 2 | Backend core (migraciones + servicios) |
| 3 | APIs + uploads + tests backend |
| 4 | Frontend wizard + gestión básica |
| 5 | Previsualización + tabla dinámica + clientes |
| 6 | PDFs + emails + cron jobs |
| 7 | Hardening, documentación y despliegue |

Total aproximado: 7 semanas desde el kickoff real (puede comprimirse si se paraleliza equipo).

---

### Dependencias y riesgos
- **Facturas de referencia**: bloqueante para maqueta visual y plantilla PDF.
- **Hosting objetivo**: define pipelines y posibles límites (memoria, workers, Node disponible).
- **Branding definitivo**: colores, tipografías, logos en alta.
- **Integraciones externas** (pagos, contabilidad): planificar en roadmap futuro para no comprometer el MVP.

Mitigaciones: recopilar insumos antes de Fase 1, usar feature flags para funcionalidades en evolución, mantener configuración desde BD para ajustes rápidos.

---

### Próximos pasos inmediatos
1. Recibir las dos facturas y cualquier guideline visual.
2. Confirmar entorno de despliegue preferido.
3. Aprobar este roadmap o ajustar tiempos/prioridades.
4. Kickoff técnico (inicio Fase 1).

Con estas acciones podremos avanzar a la implementación y comenzar a construir la herramienta “obra maestra” con bases sólidas.

