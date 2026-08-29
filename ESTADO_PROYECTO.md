# Estado del Proyecto — Sistema de Gestión para Constructora

## Objetivo
Sistema web en Django para centralizar información de una empresa constructora
(obras, trabajadores, gastos, documentos). Proyecto de aprendizaje práctico de Django.
Un despliegue independiente por empresa cliente (NO multi-tenant).

## Decisiones de arquitectura tomadas

- **No multi-tenant**: cada constructora tiene su propio despliegue. El "template"
  se reutiliza a nivel de código, no de base de datos compartida.
- **Control de acceso: Rol + Alcance**, no jerarquía estricta de cadena de mando.
  - Dueño → alcance: toda la empresa.
  - Jefe de área (Contabilidad, RRHH, Bodega, IT) → alcance: transversal a su área,
    todas las obras.
  - Administrador de obra → alcance: total sobre su obra específica.
  - Trabajador → alcance: su propio perfil (acceso al sistema opcional).
- **Usuario ≠ Trabajador**: modelos separados, relacionados opcionalmente.
  Usuario = credenciales de acceso. Trabajador = perfil/persona (con o sin acceso).
- **AsignacionObra** (Trabajador↔Obra) es un modelo "through" explícito, no un
  ManyToMany simple — necesita cargo operacional, fechas de inicio/fin.
- **Trabajador tiene su propia app** (no vive dentro de `obras`), porque puede
  existir independientemente de una obra (RRHH, Dueño, jefes de área, etc.).

## Estructura del proyecto
constructora/
├── config/ # settings, urls raíz
├── apps/
│ ├── accounts/ # Usuario, autenticación, roles/permisos
│ ├── trabajadores/ # Trabajador (perfil, sin depender de obra)
│ ├── obras/ # Obra, AsignacionObra
│ ├── gastos/ # Gasto (simple, MVP)
│ └── documentos/ # Documento
├── manage.py


## MVP definido
Usuarios → Obras → Trabajadores/Asignación → Gastos (simple) → Documentos (simple) → Dashboard

## Backlog v2 (NO implementar aún)
- EPP / registro de prevención de riesgos por obra
- Bodega descentralizada por obra con trazabilidad de movimientos/transferencias
- RRHH local por obra: horas trabajadas, permisos/licencias
- Flujo completo de Adquisiciones (Requerimiento → Cotización → OC → Despacho → Recepción)
- Contabilidad formal / centros de costo / comparación presupuesto vs. real
- Módulo de Contratistas como entidad separada

## Preguntas de negocio aún abiertas
- Flujo exacto de aprobación de gastos fuera del contexto de una obra
  (ej. compra de IT) — por ahora: Contabilidad aprueba, a definir con más detalle.

## Progreso — Etapas completadas

### Etapa 0: Estructura del proyecto 
- Proyecto Django (v6.1) creado con `django-admin startproject config .`
- Estructura `config/` + `apps/` implementada
- 5 apps creadas con `startapp <nombre> .\apps\<nombre>`
- Apps registradas en INSTALLED_APPS como `apps.<nombre>`
- `apps/__init__.py` agregado explícitamente (evitar namespace package implícito)
- **Lección aprendida**: al mover apps dentro de `apps/`, el atributo `name` en
  cada `apps.py` (ej. `TrabajadoresConfig.name`) debe coincidir EXACTAMENTE con
  la ruta declarada en INSTALLED_APPS (`apps.trabajadores`, no `trabajadores`).
  `startapp` no lo hace automático porque asume que la app vive en la raíz.

## Próximo paso
Etapa 1: Modelos — empezar por `Obra` (entidad más simple para introducir
models.py, migrations, ORM básico y Django admin).