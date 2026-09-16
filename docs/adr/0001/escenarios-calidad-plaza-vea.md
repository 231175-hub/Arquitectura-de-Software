# Escenarios de calidad — Plaza Vea (Supermercados Peruanos S.A.)

Formato de 6 partes: fuente del estímulo, estímulo, artefacto, entorno, respuesta,
medida de respuesta.

---

## Escenario 1 — Disponibilidad

| Campo | Descripción |
|---|---|
| **Fuente del estímulo** | Falla no planificada del proveedor SaaS de e-commerce (VTEX) o de una zona de disponibilidad en Google Cloud |
| **Estímulo** | Caída total o parcial de la plataforma de e-commerce durante horario de operación normal |
| **Artefacto** | Plataforma de e-commerce (VTEX) y capa de microservicios de integración |
| **Entorno** | Operación normal, horario pico de ventas (no una campaña masiva) |
| **Respuesta** | El sistema detecta la caída, informa al usuario con un mensaje claro, y preserva sin pérdida los pedidos y carritos en curso al momento de la falla; los canales alternos (tienda física, self-checkout) siguen operativos porque no dependen de la disponibilidad del e-commerce |
| **Medida de respuesta** | Tiempo de detección de la falla menor a 5 minutos; ningún pedido confirmado antes de la caída se pierde (RPO = 0 para transacciones ya confirmadas); tiempo de recuperación del servicio (RTO) menor a 2 horas |

---

## Escenario 2 — Rendimiento (Performance)

| Campo | Descripción |
|---|---|
| **Fuente del estímulo** | Una venta se registra en el punto de venta (POS) de una tienda física |
| **Estímulo** | El sistema debe reflejar el cambio de stock resultante en el catálogo del e-commerce |
| **Artefacto** | Pipeline de captura de cambios (CDC) sobre la base de datos de SAP, publicado vía Kafka hacia el catálogo de VTEX |
| **Entorno** | Operación normal, sin incidentes de red ni sobrecarga del broker de eventos |
| **Respuesta** | El evento de cambio de stock se captura, se publica en el tópico correspondiente y actualiza el catálogo digital sin intervención manual |
| **Medida de respuesta** | La actualización de stock se refleja en el catálogo online en menos de 30 segundos desde el momento de la venta en tienda, en al menos el 99% de los casos |

---

## Escenario 3 — Mantenibilidad

| Campo | Descripción |
|---|---|
| **Fuente del estímulo** | El equipo de desarrollo / producto |
| **Estímulo** | Se requiere incorporar un nuevo canal de venta (ej. un marketplace externo) sin modificar el núcleo transaccional de SAP |
| **Artefacto** | Capa de microservicios y APIs de integración construida sobre el bus de eventos (Kafka) |
| **Entorno** | Ciclo de desarrollo normal, fuera de producción crítica (ambiente de staging) |
| **Respuesta** | El nuevo canal se integra como un nuevo consumidor de los eventos ya existentes en Kafka, sin modificar el código del núcleo SAP ni el de otros servicios en producción |
| **Medida de respuesta** | El nuevo canal queda integrado y desplegado en menos de 3 semanas de esfuerzo de un equipo pequeño (2–3 desarrolladores), con cero cambios requeridos en el núcleo SAP |

---

## Escenario 4 — Seguridad

| Campo | Descripción |
|---|---|
| **Fuente del estímulo** | Un miembro autorizado del equipo de pricing, ya sea por error humano o intento malicioso |
| **Estímulo** | Se intenta publicar en producción un cambio de precio con un descuento anómalo (mayor al umbral definido, ej. 50% sobre el precio de lista) fuera de una promoción planificada |
| **Artefacto** | Módulo de gestión de precios y promociones dentro de VTEX, junto con una capa de validación de cambios |
| **Entorno** | Operación normal, sin campaña de descuentos activa que justifique el cambio |
| **Respuesta** | El sistema retiene el cambio antes de publicarlo en el sitio en vivo, exige una segunda aprobación de un supervisor, y genera una alerta automática al equipo de control interno |
| **Medida de respuesta** | El 100% de los cambios de precio que excedan el umbral de descuento definido quedan bloqueados para aprobación manual antes de publicarse; ningún cambio anómalo llega a producción sin una segunda validación registrada |

---

## Nota de trazabilidad

El escenario 4 está inspirado directamente en el incidente de precios de mayo de 2021
documentado en una resolución de Indecopi (Res. Final 084-2022/CC3), donde un error de
configuración sin segunda validación permitió que productos de alto valor se publicaran
con un descuento del 99%. Los escenarios 1–3 son inferencias razonadas a partir de la
arquitectura descrita en fuentes públicas (VTEX como SaaS, pipeline CDC + Kafka,
migración hacia microservicios); no son datos confirmados por la empresa.
