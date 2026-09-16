# ADR-0001: Adoptar arquitectura híbrida SOA/microservicios sobre el núcleo transaccional SAP

**Estado:** Aceptado (reconstrucción académica)
**Fecha:** 2019-06-01 (fecha aproximada e inferida — no confirmada por la empresa)
**Decisores:** Equipo de Arquitectura de TI, Supermercados Peruanos S.A. (reconstrucción para ejercicio académico)
**Contexto técnico relacionado:** Plataforma de e-commerce y canales digitales (SAP + VTEX + Kafka)

---

## Contexto y problema

Supermercados Peruanos S.A. opera desde 2009 un núcleo transaccional monolítico
basado en SAP, que sostiene inventario, finanzas y logística de todas las
tiendas. El negocio necesita expandir sus canales digitales (e-commerce,
marketplace, integraciones con proveedores) a un ritmo mucho más rápido del
que el monolito permite modificar de forma segura, sin detener la operación
crítica de tiendas y logística.

¿Qué estilo arquitectónico debe adoptar el proyecto digital para crecer sin
comprometer la estabilidad del núcleo transaccional existente?

## Fuerzas / restricciones en juego

* Se necesita lanzar y modificar canales digitales con ciclos de entrega
  cortos, sin depender del ciclo de cambio del ERP.
* El núcleo SAP es crítico para la operación (inventario, finanzas) y no
  puede reescribirse ni reemplazarse sin riesgo operativo significativo.
* El equipo de desarrollo es limitado frente al tamaño del negocio, y no
  puede mantener dos sistemas de registro de inventario en paralelo sin
  generar inconsistencias.
* Existe presión de negocio por tiempo de salida al mercado (time-to-market)
  para nuevas capacidades digitales.

## Opciones consideradas

1. **Statu quo: todo dentro del monolito SAP**

   * + Cero riesgo de integración adicional; un solo sistema que mantener.
   * − Cada cambio digital compite por el mismo ciclo de despliegue del
     núcleo crítico, haciendo lento y riesgoso cualquier lanzamiento.

2. **Reescritura completa ("big bang") a microservicios**

   * + Arquitectura limpia y homogénea desde el inicio, sin deuda técnica
     de transición.
   * − Alto riesgo de interrumpir la operación durante la migración; exige
     detener gran parte del desarrollo de nuevas funciones mientras dura
     la reescritura.

3. **Arquitectura híbrida incremental (estilo "strangler fig")**

   * + Riesgo distribuido en el tiempo; entrega de valor continua; permite
     usar una plataforma SaaS ya madura (VTEX) para el canal de e-commerce
     en vez de construir un motor de comercio propio.
   * − Complejidad de mantener dos paradigmas (monolito + microservicios)
     de forma simultánea durante la transición.

## Decisión

Se adopta una **arquitectura híbrida incremental**: SAP se mantiene como
sistema de registro para inventario, finanzas y logística; se adopta
**VTEX** como plataforma SaaS para el canal de e-commerce; y ambos se
conectan mediante un puente de eventos basado en **captura de cambios (CDC)
publicado vía Kafka**, que propaga el estado del inventario del núcleo hacia
el canal digital casi en tiempo real. Nuevas capacidades (integraciones,
marketplace, pricing) se construyen como servicios independientes que
consumen esos eventos, en lugar de agregarse al monolito.

## Justificación

Esta opción reduce el riesgo operativo porque el núcleo crítico no se toca
mientras se moderniza el resto del sistema, acelera el tiempo de entrega de
nuevas capacidades digitales al no depender del ciclo de cambio del ERP, y
aprovecha la experiencia de un proveedor especializado (VTEX) para el motor
de comercio en vez de construir y mantener uno propio desde cero.

## Consecuencias

**Positivas**

* Se reduce el riesgo operativo al no tocar el núcleo crítico durante la
  modernización.
* Se acelera la entrega de nuevas capacidades digitales.
* Se delega en un proveedor especializado (VTEX) el mantenimiento del motor
  de comercio y buena parte del cumplimiento de seguridad de pagos (PCI-DSS).

**Negativas / riesgos**

* Coexisten dos paradigmas arquitectónicos (monolito + microservicios)
  durante un período prolongado, aumentando la complejidad operativa y de
  aprendizaje del equipo.
* La consistencia entre el núcleo y el canal digital depende de la
  confiabilidad del puente de eventos (CDC + Kafka); una falla ahí puede
  generar inventario desincronizado.
* Se introduce dependencia de un proveedor externo (VTEX) para una función
  de negocio central, con riesgo de disponibilidad y de portabilidad.

**Seguimiento**

* Revisar en 12 meses si conviene seguir extrayendo capacidades del
  monolito SAP hacia microservicios, o si conviene evaluar alternativas a
  VTEX.
* Evaluar si el puente CDC + Kafka necesita una capa de validación de
  reglas de negocio (por ejemplo, límites de descuento) antes de propagar
  cambios sensibles como precios a producción.

## Referencias

* Resolución Final 084-2022/CC3 de Indecopi (confirma el uso de VTEX como
  plataforma de e-commerce).
* Perfil profesional público (LinkedIn) de un ex-integrante del equipo de
  integración de Supermercados Peruanos, que describe el uso de Kafka, CDC
  y Apache Airflow.
* ADR relacionado: ninguno aún — este es el primero.
* Nota: este documento es una reconstrucción académica a partir de
  evidencia pública; no es un ADR oficial de Supermercados Peruanos S.A.
