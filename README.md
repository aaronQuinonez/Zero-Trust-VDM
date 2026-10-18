# Sistema de Control de Acceso Zero Trust formalizado con VDM++

Proyecto del curso **Aspectos Formales de Especificación y Verificación** — Escuela Profesional de Ingeniería de Sistemas, Universidad Nacional de San Agustín de Arequipa (UNSA).

## Descripción
El proyecto propone modelar y verificar formalmente un motor de autorización (*Policy Engine*) basado en la arquitectura **Zero Trust** utilizando **VDM++**. El sistema evalúa solicitudes de acceso considerando identidad, rol, estado del dispositivo, criticidad del recurso, verificación MFA y nivel de riesgo contextual para tomar decisiones de acceso (`AUTORIZADO`, `DENEGADO`, `REVOCADO`) y evitar estados de autorización inválidos.

## Integrantes
* Alvarez Llave, Fabian Andre
* Condori Caira, Antony Beltran
* Quiñonez Delgado, Aaron Fernando
* Sencia Ale, Bryan Daniel
* Yauli Merma, Diego Raul

## Estructura del Repositorio
* `model/`: Especificaciones formales en VDM++ (`.vpp`) y casos de prueba.
* `docs/`: Documentación justificativa por etapa, diagramas PlantUML y referencias.
* `entregables/`: Reportes de cobertura (`tcov`/`rtinfo`) y material de presentación.

## Plan de Desarrollo Progresivo
- [x] **E0:** Definir alcance, estructura de carpetas y `README.md`.
- [x] **E1:** Identificación de tipos `Quote` y `Token` (`model/Tipos.vpp`).
- [x] **E2:** Registro e invariante para una solicitud de acceso (`model/SolicitudAcceso.vpp`).
- [ ] **E3:** Uso de `Seq` (historial) y `Set` (permisos) (`model/Usuario.vpp`).
- [ ] **E4:** Diagrama de clases inicial (`docs/clases.puml` y `model/Recurso.vpp`).
- [ ] **E5:** Uso del tipo `Map` para asociar entidades y permisos (`model/MotorZeroTrust.vpp`).
- [ ] **E6:** Invariantes globales del sistema (`model/MotorZeroTrust.vpp`).
- [ ] **E7:** Operaciones del motor Zero Trust: evaluar, autorizar, denegar y revocar (`model/MotorZeroTrust.vpp`).
- [ ] **E8:** Casos de prueba y análisis de cobertura (`model/Pruebas.vpp`).
- [ ] **E9:** Estados, transiciones y propiedades críticas para *Model Checking*.
