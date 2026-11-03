# Taller Greenfield con Spec Kit

Este repositorio reúne los materiales de un taller para diseñar contratos de API Open Finance desde cero (*greenfield*), utilizando Spec Kit como apoyo al trabajo de especificación. El objetivo es convertir requisitos funcionales aprobados en contratos OpenAPI modulares, trazables y verificables.

El taller se centra en el diseño del contrato. No genera ni despliega servicios, controladores, bases de datos ni código de backend. El contrato consolidado y validado queda preparado para una posible pipeline posterior de publicación en un gestor de APIs, que es un flujo independiente.

## Caso práctico

El ejercicio principal diseña `of-prc-port-investment-summary-v1`, una Process API que expone un resumen de inversiones. En el contexto arquitectónico, una Experience API presenta ese resumen y las capacidades de cartera y cuentas proceden de System APIs. Estas relaciones sirven para analizar el diseño: no implican que las APIs o sus integraciones estén implementadas ni que siempre deban existir las tres capas.

Antes de generar el contrato, el equipo acuerda las operaciones, rutas, parámetros, datos, reglas de negocio, seguridad, errores y criterios de aceptación. No se inventan requisitos para completar decisiones pendientes.

## Recorrido del taller

1. Preparar el gobierno aplicable: constituciones corporativa y de dominio, perfil de la API y constitución efectiva que utilizará Spec Kit.
2. Especificar las APIs, operaciones, modelos canónicos y criterios de aceptación; revisar la comunicación con las 7 C.
3. Resolver ambigüedades con las personas responsables antes de diseñar el contrato.
4. Planificar los proyectos y artefactos contractuales, las referencias compartidas y las comprobaciones necesarias.
5. Generar un contrato OpenAPI modular por API, reutilizando los componentes aprobados.
6. Validar el contrato, consolidarlo en un único documento por API y revisar la trazabilidad con los requisitos.
7. Presentar los artefactos y evidencias para su revisión y aprobación.

## Resultados esperados

- Especificación funcional y decisiones de diseño acordadas.
- Matriz de trazabilidad entre requisitos, APIs, operaciones, modelos y criterios de aceptación.
- Contrato modular con documento raíz y recursos, parámetros y esquemas referenciados.
- Referencias a los componentes comunes y modelos canónicos aprobados, con sus revisiones fijadas.
- Informes de validación y contrato consolidado, asociado a la revisión y versión correspondientes.

## Enfoque y límites

- La especificación debe ser suficientemente clara y completa antes de generar artefactos. Las decisiones relevantes pendientes se convierten en preguntas, no en supuestos.
- La arquitectura API-Led guía la asignación de responsabilidades; no se crean capas automáticamente.
- Se conserva la trazabilidad de requisitos, modelos y decisiones de gobierno.
- El contrato modular es la fuente de verdad; el consolidado es un artefacto generado y no se edita manualmente.
- La validación y consolidación se mantienen separadas de la publicación en el gestor de APIs.
- Las secciones sobre Jenkins, composición de constituciones y recursos de Copilot describen un flujo propuesto para el taller. No afirman que esas automatizaciones estén implementadas.

## Materiales

- [Guía del taller](docs/taller-scpeckit-greenfield.md): alcance, arquitectura, convenciones, estructura contractual, flujo de Spec Kit y validación.
- [Anexo de constituciones](docs/anexo%20Construcci%C3%B3n%20y%20especializaci%C3%B3n%20de%20constituciones.md): composición del gobierno corporativo, de dominio y de API, más la propuesta de pipeline Jenkins.
- [Especificaciones de ejemplo](docs/especificaciones-resumen-inversiones/): documentos funcionales y ejemplos relacionados con el resumen de inversiones y sus APIs.

## Preparación

El taller requiere un repositorio de trabajo, acceso a GitHub Copilot con los flujos de Spec Kit disponibles, convenciones de diseño y modelos compartidos aprobados. Para validar contratos se contemplan Stoplight Spectral y Redocly CLI con versiones fijadas y compatibles. La disponibilidad de GitLab, Jenkins, permisos y automatizaciones debe comprobarse y acordarse en el entorno donde se realice el ejercicio.