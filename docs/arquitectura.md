## 1. Visión general

Herramienta SaaS para crear y gestionar presupuestos y facturas desde cualquier dispositivo. Debe permitir configurar logos, numeraciones, textos legales y plantillas, además de generar PDFs profesionales listos para enviarse por email o firmarse.

## 2. Objetivos clave

- Experiencia ultra sencilla: en menos de 5 pasos se genera un documento listo para enviar.
- Configurable al 100 %: logos, numeración, moneda, impuestos, textos legales y plantillas.
- Escalable: multiempresa, multiusuario y con APIs limpias para futuras integraciones (pagos, CRM, etc.).
- Robusto: validación fuerte, logs, colas para emails, control de estados y trazabilidad.

## 3. Experiencia de usuario (UX)

1. Pantalla de bienvenida con dos CTAs grandes: **Crear presupuesto** y **Crear factura**.
2. Asistente en pasos:
   - Empresa (se puede elegir una existente o crear otra).
   - Cliente (buscar/crear cliente).
   - Cabecera del documento (número, fechas, condiciones de pago, notas internas).
   - Líneas dinámicas (servicio/producto, cantidad, precio, IVA, descuentos por línea).
   - Logo y previsualización en vivo.
3. Acciones posteriores:
   - Guardar borrador.
   - Generar PDF.
   - Enviar por email y registrar eventos (enviado, aceptado, pagado).
   - Convertir presupuesto en factura manteniendo histórial.

## 4. Módulos funcionales

- **Empresas**: datos fiscales, logos, banca, textos legales, plantillas preferidas.
- **Clientes**: fichas con contactos, direcciones, etiquetas.
- **Documentos**:
  - Tipos: factura, presupuesto (futuros: albarán, proforma).
  - Estados configurables: borrador, enviado, aceptado, rechazado, pagado, vencido.
  - Conversión de presupuesto a factura.
- **Líneas**: productos/servicios reutilizables, impuestos configurables, descuentos.
- **Configuración global**:
  - Numeradores por tipo/documento/año.
  - Monedas, tipos de IVA predefinidos, plantillas PDF.
  - Textos legales, condiciones de pago estándar, cuentas bancarias.
- **Gestión de logos**:
  - Logo por defecto por empresa.
  - Drag & Drop para reemplazar logo puntual.
  - Historial de logos usados por documento (para trazabilidad).
- **Generación y distribución de PDFs**:
  - Plantillas HTML responsivas con estilos corporativos.
  - Render en backend via Snappy (wkhtmltopdf) o Browsershot.
  - Almacenamiento en S3-compatible y enlaces firmados para descargas.
- **Notificaciones**:
  - Toasts en frontend.
  - Emails transaccionales (envío de factura, recordatorios de vencimiento).
  - Webhooks/API para futuras integraciones.

## 5. Requisitos no funcionales

- **Disponibilidad**: 99.5 %, con backups automáticos de BD y archivos.
- **Seguridad**: TLS, CSRF, sanitización, rol-based access, logs auditables.
- **Rendimiento**: respuestas < 200 ms en endpoints principales con caching selectivo.
- **Escalabilidad**: arquitectura desacoplada front/back, soporte horizontal vía contenedores.
- **Internacionalización**: textos y formatos listos para traducirse; locale configurable por empresa.

## 6. Arquitectura técnica

### 6.1 Frontend
- React + Vite.
- State management con Zustand o Redux Toolkit.
- TailwindCSS + componentes propios (botones, inputs, tablas, cards).
- React Query para consumo de API y optimismo en formularios.
- Previsualización PDF via iframe/html renderer con datos en vivo.

### 6.2 Backend
- Laravel 11 + PHP 8.3.
- Controllers REST (API Resource).
- Servicios de dominio para numeración, impuestos, generación PDF.
- Jobs/queues (Redis) para emails y exportaciones masivas.
- Events + listeners para auditoría y webhooks.
- Sanctum para auth SPA, Passport opcional a futuro.

### 6.3 Infraestructura
- Hosting compartido: deploy monolítico (Laravel + build React servida desde `public/`).
- VPS/Docker (DigitalOcean/Hetzner): nginx + php-fpm + queue worker + cron + MySQL/PostgreSQL + storage S3 compatible.
- Observabilidad: Laravel Telescope + Monolog (Papertrail) + métricas Prometheus/Grafana opcionales.

## 7. Modelo de datos inicial

| Tabla | Campos claves |
|-------|---------------|
| `companies` | nombre, cif, dirección, logo_id, preferencias JSON |
| `clients` | company_id, nombre, cif, dirección, email, tags |
| `documents` | company_id, client_id, tipo (`invoice`, `quote`), número, fechas, estado, moneda, totales, impuestos, notas, logo_override_id |
| `document_items` | document_id, orden, descripción, cantidad, precio_unitario, impuesto_id, descuento_pct, subtotal |
| `taxes` | nombre, porcentaje, tipo, por defecto |
| `settings` | clave, valor, scope (global/empresa) |
| `media` | ruta, tipo, metadata (para logos, firmas) |
| `users`, `roles`, `permissions` | auth y control de acceso |
| `activities` | historial de acciones (creado, enviado, pagado) |

> El modelo se ajustará con las facturas reales para asegurar todos los campos necesarios (formas de pago, IBAN, referencias, etc.).

## 8. API REST (principales endpoints)

- `POST /auth/login`, `POST /auth/logout`, `GET /me`
- `GET/POST/PATCH /companies`, `/clients`
- `GET/POST/PATCH /documents`
  - `POST /documents/{id}/items`
  - `POST /documents/{id}/status`
  - `POST /documents/{id}/convert` (presupuesto → factura)
- `POST /documents/{id}/pdf` (genera y retorna URL)
- `POST /documents/{id}/send-email`
- `POST /logos` (upload drag & drop), `DELETE /logos/{id}`
- `GET/PUT /settings`

Todas las respuestas siguen JSON:API, con validaciones server-side y mensajes claros.

## 9. Flujo de generación de PDF

1. El usuario previsualiza el template en frontend (HTML).
2. Al guardar el documento, se envía payload limpio al backend.
3. El backend renderiza la plantilla Blade específica (factura/presupuesto) y la convierte a PDF via Snappy/Browsershot.
4. El PDF se guarda en storage (`documents/{año}/{numero}.pdf`).
5. Se devuelve URL firmada; también se adjunta si se envía por email.
6. Al modificar datos relevantes, se invalida el PDF y se regenera bajo demanda.

## 10. Gestión de logos y assets

- Logo por defecto definido en `settings`.
- Área drag & drop en frontend con preview inmediata.
- Upload firmado al backend (limitado a formatos PNG/SVG/JPG, máx. 2 MB).
- Las versiones de logo usadas quedan asociadas al documento para evitar cambios retroactivos.

## 11. Seguridad y cumplimiento

- Auth vía Laravel Sanctum, renovación de tokens, MFA opcional futura.
- Roles: admin (gestiona todo), manager (documentos), viewer (solo lectura).
- Campos sensibles cifrados (`encrypt` de Laravel) cuando aplica.
- Registros auditados en `activities` y exported a logs externos.
- Cumplimiento RGPD: consentimiento para almacenar datos de clientes, posibilidad de anonimizar/borrar bajo petición.

## 12. Observabilidad y mantenimiento

- Monitoreo de colas y jobs fallidos.
- Alertas por email/Slack cuando una cola supere umbrales o una tarea crítica falle.
- Backups diarios automatizados (base de datos y storage).
- Scripts de mantenimiento (`artisan schedule:run`) para limpieza de borradores antiguos, regeneración de numeradores, recordatorios de vencimientos.

## 13. Pendientes inmediatos

- Recibir las dos facturas de referencia para ajustar:
  - Campos obligatorios.
  - Distribución visual y tipografías.
  - Textos legales y firmas.
- Validar con el cliente:
  - Hosting objetivo (compartido vs VPS).
  - Idiomas y monedas requeridas desde el día uno.
  - Necesidades de multiempresa/multiusuario.

Con esta base podemos comenzar a levantar el repositorio (Laravel + React) y crear las primeras migraciones, seeders y componentes UI.

