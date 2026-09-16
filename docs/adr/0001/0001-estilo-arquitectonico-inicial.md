<!-- Ubicación sugerida en el repo: /docs/adr/0001-estilo-arquitectonico-inicial.md -->

# 1. Estilo arquitectónico inicial del proyecto: híbrido SOA/microservicios sobre el núcleo transaccional existente

## Status

Aceptada

## Context and Problem Statement

Supermercados Peruanos S.A. opera un núcleo transaccional monolítico basado en SAP
(ERP) que sostiene inventario, finanzas y logística desde 2009. El negocio necesita
expandir canales digitales (e-commerce, marketplace, integraciones con proveedores)
a un ritmo mucho más rápido del que el monolito permite modificar de forma segura,
y sin detener la operación crítica de tiendas y logística.

¿Qué estilo arquitectónico debe adoptar el proyecto para soportar el crecimiento
digital sin comprometer la estabilidad del núcleo transaccional existente?

## Decision Drivers

* Necesidad de lanzar y modificar canales digitales (e-commerce, marketplace) con
  ciclos de entrega cortos, sin depender de los ciclos de cambio del ERP.
* El núcleo SAP es crítico para la operación (inventario, finanzas) y no puede
  reescribirse ni reemplazarse sin riesgo operativo significativo.
* El equipo de desarrollo es limitado y no puede mantener dos sistemas de registro
  de inventario en paralelo sin generar inconsistencias.
* Existe presión de negocio por tiempo de salida al mercado (time-to-market) para
  nuevas capacidades digitales.

## Considered Options

* **Opción A — Statu quo:** mantener todo el desarrollo dentro del monolito SAP,
  incluyendo la lógica de e-commerce y canales digitales.
* **Opción B — Reescritura completa ("big bang"):** migrar todo el sistema a una
  arquitectura de microservicios de una sola vez, reemplazando el núcleo SAP.
* **Opción C — Arquitectura híbrida incremental (estilo "strangler fig"):** mantener
  SAP como sistema de registro para inventario/finanzas, y extraer capacidades
  digitales específicas hacia servicios independientes y una plataforma SaaS externa
  para el canal de e-commerce, conectados mediante un puente de eventos.

## Decision Outcome

Opción elegida: **"Arquitectura híbrida incremental (estilo strangler fig)"**,
porque permite modernizar los canales digitales sin detener ni arriesgar la
operación crítica que sostiene el núcleo SAP, y porque el equipo puede entregar
valor de forma incremental en vez de asumir el riesgo de una reescritura total.

En concreto, esto implica:

* Mantener SAP como fuente de verdad para inventario, finanzas y logística.
* Adoptar una plataforma SaaS de e-commerce (VTEX) en vez de construir un motor de
  comercio propio desde cero.
* Construir un puente de sincronización basado en captura de cambios (CDC) publicado
  vía un bus de eventos (Kafka), para propagar el estado del inventario del núcleo
  hacia el canal digital casi en tiempo real.
* Extraer nuevas capacidades (integraciones, marketplace, pricing) como servicios
  independientes que consumen esos eventos, en vez de agregarlas al monolito.

### Consequences

**Buenas**

* Se reduce el riesgo operativo: el núcleo crítico no se toca mientras se moderniza
  el resto del sistema.
* Se acelera el tiempo de entrega de nuevas capacidades digitales al no depender del
  ciclo de cambio del ERP.
* Se aprovecha la experiencia y el mantenimiento de un proveedor especializado
  (VTEX) para el motor de comercio, en vez de construir y mantener uno propio.

**Malas**

* Coexisten dos paradigmas arquitectónicos (monolito + microservicios) durante un
  período prolongado, lo que aumenta la complejidad operativa y de aprendizaje del
  equipo.
* La consistencia entre el núcleo y el canal digital depende de la confiabilidad del
  puente de eventos (CDC + Kafka); una falla ahí puede generar inventario
  desincronizado.
* Se introduce dependencia de un proveedor externo (VTEX) para una función de
  negocio central como el e-commerce, con el riesgo de disponibilidad y de
  portabilidad que eso conlleva.

## Pros and Cons of the Options

### Opción A — Statu quo

* Bueno: cero riesgo de integración adicional, un solo sistema que mantener.
* Malo: cada cambio digital compite por el mismo ciclo de despliegue del núcleo
  crítico, haciendo lento e riesgoso cualquier lanzamiento.

### Opción B — Reescritura completa

* Bueno: arquitectura limpia y homogénea desde el inicio, sin deuda técnica de
  transición.
* Malo: alto riesgo de interrumpir la operación durante la migración; exige
  detener gran parte del desarrollo de nuevas funciones mientras dura la reescritura.

### Opción C — Híbrida incremental (elegida)

* Bueno: riesgo distribuido en el tiempo, entrega de valor continua, aprovecha
  soluciones ya maduras del mercado (VTEX).
* Malo: complejidad de mantener dos paradigmas simultáneamente durante la
  transición; requiere disciplina para no acumular integraciones ad-hoc.

## More Information

Esta decisión es una reconstrucción razonada a partir de evidencia pública (uso
confirmado de VTEX según resolución de Indecopi 084-2022/CC3, y uso de Kafka/CDC/
Airflow según el historial profesional público de un ex-integrante del equipo de
integración de la empresa). No es un documento oficial de Supermercados Peruanos
S.A.
