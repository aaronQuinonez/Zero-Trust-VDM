# Etapas E3, E5, E6 y E7: Uso de Seq, Set, Map, Invariantes Globales y Operaciones

## 1. Uso de `Seq` y `Set` (E3)

En el sistema Zero Trust se utilizan ambos tipos de colección.

El tipo `Seq` se utiliza en `MotorZeroTrust` para almacenar el historial de solicitudes de acceso:

    private historial : seq of SolicitudAcceso`SolicitudAcceso := [];

Se eligió una secuencia porque el orden temporal de las solicitudes es importante. De esta manera, se puede conservar el orden en que fueron evaluadas.

El tipo `Set` se utiliza para representar los recursos permitidos para cada usuario (tanto en `Usuario` como en `MotorZeroTrust`):

    set of RecursoId

Se utiliza un conjunto porque el orden de los recursos no es importante y no se requiere almacenar elementos repetidos.

---

## 2. Uso de `Map` (E5)

El sistema utiliza tres mapas en `MotorZeroTrust` para relacionar identificadores con las entidades principales del proyecto:

    private usuarios : map UsuarioId to Usuario := {|->};

Relaciona cada identificador de usuario con su objeto `Usuario`.

    private recursos : map RecursoId to Recurso := {|->};

Relaciona cada identificador de recurso con su objeto `Recurso`.

    private permisos : map UsuarioId to set of RecursoId := {|->};

Relaciona cada usuario con el conjunto de recursos a los que tiene permiso de acceso.

Por ejemplo:

`U001 |-> {R001, R002}`

indica que el usuario `U001` tiene permisos sobre los recursos `R001` y `R002`.

---

## 3. Invariantes Globales (E6)

En `model/MotorZeroTrust.vpp` se definen dos invariantes combinadas para mantener la integridad referencial del sistema:

    inv
      (dom permisos subset dom usuarios) and
      (forall uid in set dom permisos & permisos(uid) subset dom recursos);

La primera condición (`dom permisos subset dom usuarios`) garantiza que todo usuario que tenga permisos asignados se encuentre previamente registrado en el sistema.

La segunda condición (`forall uid in set dom permisos & permisos(uid) subset dom recursos`) garantiza que todos los recursos asignados como permisos existan previamente en el mapa de recursos.

De esta manera, se evita que existan permisos asociados a usuarios o recursos inexistentes.

---

## 4. Operaciones del Motor (`MotorZeroTrust.vpp`)

### 4.1 Operaciones administrativas (E5–E6)
Para gestionar los mapas se implementan las siguientes operaciones:

- `RegistrarUsuario(u : Usuario)` — registra un usuario nuevo (`pre u.GetId() not in set dom usuarios`).
- `RegistrarRecurso(r : Recurso)` — registra un recurso nuevo (`pre r.GetId() not in set dom recursos`).
- `ConcederPermiso(uid : UsuarioId, rid : RecursoId)` — asigna un recurso permitido a un usuario (`pre uid in set dom usuarios and rid in set dom recursos`).

### 4.2 Operaciones dinámicas de autorización y revocación (E7 — Implementadas)
Las operaciones dinámicas de la etapa E7 ya están implementadas en `model/MotorZeroTrust.vpp`:

- `EvaluarAcceso(sol : SolicitudAcceso`SolicitudAcceso) : EstadoAcceso` — evalúa una solicitud nueva (`pre forall s in set elems historial & s.id <> sol.id`). Devuelve `<AUTORIZADO>` únicamente si el usuario está registrado y activo, el recurso existe, el usuario posee permiso explícito sobre el recurso, el dispositivo es válido (`dispositivoValido`), el riesgo es menor a 80 (`riesgo < 80`) y, en caso de recurso crítico, se verificó MFA (`mfaVerificado`); en caso contrario devuelve `<DENEGADO>`. Registra la solicitud actualizada con su decisión al final de `historial`. *(Nota: Aunque `Usuario` incluye el atributo `rol`, `EvaluarAcceso` no lo consulta directamente en la versión actual).*
- `RevocarAcceso(solId : SolicitudId) : bool` — recorre `historial` (`pre len historial > 0`) y, si encuentra una solicitud con `id = solId` cuyo estado sea `<AUTORIZADO>`, actualiza su estado a `<REVOCADO>` y devuelve `true`; en caso contrario devuelve `false`.