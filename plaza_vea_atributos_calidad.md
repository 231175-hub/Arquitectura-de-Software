# Plaza Vea (Supermercados Peruanos S.A.) — Análisis de atributos de calidad

## Historia de éxito

Plaza Vea es el e-commerce de supermercado con más tráfico de Perú (~4.2 millones de
visitas mensuales estimadas, SimilarWeb 2020) y logró extender la venta online a zonas
rurales de baja conectividad, un mercado que ningún competidor local había resuelto
técnicamente hasta entonces.

---

## 1. Atributos de calidad justificados

### Atributo 1 — Escalabilidad

El liderazgo de tráfico no es un dato aislado de marketing: está sostenido por una
arquitectura capaz de absorber ese volumen sin degradarse. VTEX (plataforma de
e-commerce SaaS confirmada mediante una resolución de Indecopi de 2022) ofrece
infraestructura elástica gestionada por el proveedor, y una capa de microservicios sobre
Kubernetes en Google Cloud está pensada explícitamente para picos de campañas masivas.
Sin esta escalabilidad, ser el sitio más visitado se traduciría en caídas constantes, no
en liderazgo sostenido.

### Atributo 2 — Rendimiento (Performance)

La segunda mitad del éxito es de alcance, no solo de volumen. El rediseño de "carga
ligera" de 2023 (72% más liviano, funcional en conexiones 2G, ilustraciones en vez de
fotos) fue lo que permitió que el negocio digital llegara a clientes en zonas rurales
antes fuera del mercado direccionable por pura incapacidad técnica de cargar el sitio.
Aquí el rendimiento no es una optimización cosmética: es la variable que decide si esos
clientes pueden comprar o no.

---

## 2. Escenarios de calidad completos

### Escenario 1 — Escalabilidad

| Campo | Descripción |
|---|---|
| **Fuente del estímulo** | Miles de usuarios simultáneos durante una campaña de venta masiva (ej. Cyber Days) |
| **Estímulo** | Un pico repentino de tráfico y transacciones concurrentes en plazavea.com |
| **Artefacto** | La plataforma de e-commerce (VTEX) y la capa de microservicios sobre Kubernetes/GCP |
| **Entorno** | Operación normal, durante una campaña planificada de alta demanda |
| **Respuesta** | El sistema escala horizontalmente (más instancias/pods) para absorber la carga adicional sin intervención manual de emergencia |
| **Medida de respuesta** | El sitio se mantiene disponible y con tiempos de respuesta aceptables durante todo el pico de tráfico, sin caídas del servicio |

### Escenario 2 — Rendimiento (Performance)

| Campo | Descripción |
|---|---|
| **Fuente del estímulo** | Un cliente ubicado en una zona rural del Perú, con conexión 2G/3G inestable |
| **Estímulo** | El cliente intenta cargar plazavea.com y completar una compra |
| **Artefacto** | La PWA (capa de presentación), específicamente su Service Worker de detección de calidad de conexión |
| **Entorno** | Condiciones de baja conectividad, fuera de las principales ciudades |
| **Respuesta** | El sitio detecta automáticamente la conexión y sirve la versión ligera (72% más liviana, ilustraciones en vez de fotos) |
| **Medida de respuesta** | El sitio carga y permite completar una compra en un tiempo razonable en 2G, en vez de fallar o quedar inutilizable |

---

## 3. Mini ADR — Arquitectura de carga adaptativa para zonas de baja conectividad

> Nota: es una reconstrucción plausible a partir de evidencia pública. Supermercados
> Peruanos nunca publicó un ADR real; se infiere como ejercicio razonado a partir del
> proyecto de "carga ligera" de 2023.

**Contexto**
El e-commerce de Plaza Vea corría sobre una PWA moderna (imágenes de alta resolución,
bundles de JavaScript completos), demasiado pesada para funcionar de forma confiable en
zonas rurales con conectividad 2G/3G. Esto no era solo un problema de experiencia de
usuario: era un techo de mercado, ya que parte de los clientes potenciales fuera de Lima
técnicamente no podían cargar el sitio lo suficientemente rápido como para comprar.

**Decisión**
Construir una arquitectura de **carga adaptativa del lado del cliente**, apoyada en el
mismo Service Worker de la PWA existente: el sitio detecta automáticamente la calidad de
conexión y, si es baja, sirve una versión ~72% más liviana (ilustraciones en vez de fotos,
elementos no críticos diferidos). El cambio es automático, no un modo manual, y se
construyó como una capa de presentación adicional sobre la misma base de VTEX, sin
duplicar catálogo ni lógica de negocio.

**Consecuencias**

*Positivas*
- Extiende el alcance del negocio digital a mercados antes inaccesibles técnicamente,
  sin mantener dos backends — solo dos vistas sobre la misma fuente de datos.
- Refuerza el rendimiento como palanca de alcance de mercado, no solo de experiencia.

*Negativas / riesgos asumidos*
- Dos experiencias visuales que probar, diseñar y mantener sincronizadas — costo de
  mantenibilidad permanente.
- La detección de calidad de conexión puede fallar en ambos sentidos (falsos positivos
  y negativos).
- Reemplazar fotos por ilustraciones puede reducir la confianza de compra en categorías
  donde ver el producto real importa (ej. frescura de alimentos).
- No existen métricas públicas e independientes sobre la mejora real de conversión en
  zonas rurales tras el cambio.

---

## Nota sobre fuentes

Este análisis combina evidencia de distinta solidez: una resolución oficial de Indecopi
(VTEX como plataforma), declaraciones públicas de una exgerente de e-commerce de la
empresa (arquitectura PWA), prensa especializada (self-checkout, carga ligera para zonas
rurales) y un perfil profesional público en LinkedIn (Kafka, CDC, Airflow, microservicios).
El ADR es una reconstrucción propia, no un documento oficial de la empresa.
