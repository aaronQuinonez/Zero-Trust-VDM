# Sistema de Control de Acceso Zero Trust formalizado con VDM++

Proyecto del curso **Aspectos Formales de Especificación y Verificación** — Escuela Profesional de Ingeniería de Sistemas, Universidad Nacional de San Agustín de Arequipa (UNSA).

## Descripción
El proyecto modela y verifica formalmente un motor de autorización (*Policy Engine*) basado en la arquitectura **Zero Trust** utilizando **VDM++** y **NuSMV**.

El motor VDM++ (`MotorZeroTrust`) evalúa solicitudes de acceso (`SolicitudAcceso`SolicitudAcceso`) comprobando que el usuario esté registrado y activo, que el recurso exista, que el usuario tenga permiso explícito sobre el recurso, que el dispositivo sea válido (`dispositivoValido`), que el riesgo contextual sea estrictamente menor a 80 (`riesgo < 80`) y que exista verificación de segundo factor (`mfaVerificado`) cuando el recurso sea crítico, emitiendo decisiones formales (`<AUTORIZADO>`, `<DENEGADO>`, `<REVOCADO>`).

> **Nota sobre el atributo `rol`:** Aunque cada instancia de `Usuario` almacena un `rol` (`<USUARIO>`, `<ADMINISTRADOR>` o `<AUDITOR>`) accesible mediante `GetRol()`, la operación `MotorZeroTrust`EvaluarAcceso` **no consulta el rol directamente en la versión actual**; la decisión se rige por la matriz explícita `permisos : map UsuarioId to set of RecursoId` y el contexto de la solicitud.

## Integrantes
* Alvarez Llave, Fabian Andre
* Condori Caira, Antony Beltran
* Quiñonez Delgado, Aaron Fernando
* Riveros Valeriano, Sebastian Alfredo
* Sencia Ale, Bryan Daniel
* Yauli Merma, Diego Raul

## Estructura del Repositorio
* [`model/`](model/): Especificaciones formales en VDM++ (`.vpp`) y suite de pruebas ([`model/Pruebas.vpp`](model/Pruebas.vpp)).
* [`model-checking/`](model-checking/): Abstracción finita en NuSMV ([`model-checking/zero_trust.smv`](model-checking/zero_trust.smv)) e instrucciones de verificación ([`model-checking/README.md`](model-checking/README.md)).
* [`docs/`](docs/): Documentación técnica y guías de uso:
  * [`docs/01-quote-token.md`](docs/01-quote-token.md): Tipos `Quote` y `Token` (E1).
  * [`docs/02-registro-invariante.md`](docs/02-registro-invariante.md): Registro `SolicitudAcceso` e invariante de riesgo (E2).
  * [`docs/03-seq-set-map.md`](docs/03-seq-set-map.md): Colecciones `Seq`, `Set`, `Map`, invariantes globales y operaciones del motor (E3, E5–E7).
  * [`docs/04-model-checking.md`](docs/04-model-checking.md): Autómata de estados, propiedades temporales y verificación en NuSMV (E9).
  * [`docs/05-pruebas-validacion.md`](docs/05-pruebas-validacion.md): Catálogo de casos de prueba, rechazos por contrato y cobertura (E8).
  * [`docs/06-guia-ejecucion.md`](docs/06-guia-ejecucion.md): Guía para abrir el proyecto, verificar sintaxis/tipos y ejecutar pruebas.
  * [`docs/clases.puml`](docs/clases.puml) / [`docs/clases.png`](docs/clases.png): Diagrama de clases actualizado del modelo.
  * [`docs/referencias.md`](docs/referencias.md): Referencias bibliográficas.
* [`entregables/`](entregables/): Reportes de cobertura ([`entregables/cobertura.E8.txt`](entregables/cobertura.E8.txt), [`entregables/cobertura_E8.png`](entregables/cobertura_E8.png)) y evidencias de ejecución de pruebas en [`entregables/E8/`](entregables/E8/).

## Plan de Desarrollo Progresivo
- [x] **E0:** Definir alcance, estructura de carpetas y `README.md`.
- [x] **E1:** Identificación de tipos `Quote` y `Token` (`model/Tipos.vpp`).
- [x] **E2:** Registro e invariante para una solicitud de acceso (`model/SolicitudAcceso.vpp`).
- [x] **E3:** Uso de `Seq` (historial) y `Set` (permisos) (`model/Usuario.vpp`).
- [x] **E4:** Diagrama de clases y modelado de entidades (`docs/clases.puml`, `model/Usuario.vpp` y `model/Recurso.vpp`).
- [x] **E5:** Uso del tipo `Map` para asociar entidades y permisos (`model/MotorZeroTrust.vpp`).
- [x] **E6:** Invariantes globales del sistema (`model/MotorZeroTrust.vpp`).
- [x] **E7:** Operaciones del motor Zero Trust: `EvaluarAcceso` y `RevocarAcceso` (`model/MotorZeroTrust.vpp`).
- [x] **E8:** Casos de prueba, validación por contrato y análisis de cobertura al 100 % ([`docs/05-pruebas-validacion.md`](docs/05-pruebas-validacion.md), [`entregables/E8/`](entregables/E8/)).
- [ ] **E9:** Estados, transiciones y verificación formal mediante *Model Checking* (modelo y resultados documentados en [`model-checking/zero_trust.smv`](model-checking/zero_trust.smv) y [`docs/04-model-checking.md`](docs/04-model-checking.md); pendiente de evidencias gráficas en `entregables/E9/`).