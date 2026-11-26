# E2 – Registro `SolicitudAcceso` e invariante de riesgo

**Archivo del modelo:** `model/SolicitudAcceso.vpp`  
**Depende de:** E1 (`model/Tipos.vpp`)

## 1. Propósito

Una *solicitud de acceso* es el evento que el motor Zero Trust debe evaluar: un usuario, desde un dispositivo, pide acceder a un recurso con cierto contexto (riesgo, MFA). En VDM++ se modela como un **registro compuesto** (`::`) dentro de la clase `SolicitudAcceso`, que agrupa campos heterogéneos y permite asociarle una **invariante** que toda instancia debe cumplir.

Dado que la clase `SolicitudAcceso` no hereda de `Tipos`, sus campos referencian los tipos de `Tipos.vpp` mediante calificación explícita (`Tipos`UsuarioId`, `Tipos`EstadoAcceso`, etc.). Asimismo, cuando otras clases (`MotorZeroTrust`, `Pruebas`) referencian este tipo registro, utilizan el nombre calificado **`SolicitudAcceso`SolicitudAcceso`**.

## 2. Descripción de campos (`model/SolicitudAcceso.vpp`)

| Campo | Tipo VDM++ | Descripción |
|---|---|---|
| `id` | `Tipos`SolicitudId` (`token`) | Identificador único de la solicitud. |
| `usuario` | `Tipos`UsuarioId` (`token`) | Usuario que solicita el acceso. |
| `recurso` | `Tipos`RecursoId` (`token`) | Recurso al que se quiere acceder. |
| `dispositivo` | `Tipos`DispositivoId` (`token`) | Dispositivo desde el que se origina la solicitud. |
| `dispositivoValido` | `bool` | `true` si el dispositivo cumple la verificación de integridad. Base de la validación de dispositivo en E7. |
| `riesgo` | `nat` | Puntaje de riesgo contextual de la solicitud (`0` a `100`). |
| `mfaVerificado` | `bool` | `true` si se verificó el segundo factor (necesario en recursos críticos). |
| `estado` | `Tipos`EstadoAcceso` (`quote`) | Estado en el ciclo de vida: `<PENDIENTE>`, `<EVALUANDO>`, `<AUTORIZADO>`, `<DENEGADO>`, `<REVOCADO>`. |

## 3. Invariante de registro

    inv s == s.riesgo <= 100;

Formalmente: para toda solicitud `s` del tipo `SolicitudAcceso`SolicitudAcceso`,

$$\text{inv } s == s.\text{riesgo} \le 100$$

### Justificación

- `riesgo` es `nat`, que ya garantiza `riesgo >= 0`; solo hace falta acotar el límite superior.
- El riesgo se interpreta como una escala de 0 a 100; un valor mayor no tiene significado en el dominio.
- Al estar en el **tipo**, la invariante se comprueba cada vez que se construye (`mk_SolicitudAcceso`SolicitudAcceso`) o modifica (`mu`) una `SolicitudAcceso`SolicitudAcceso`: una solicitud inválida no puede existir en el modelo. Esto evita validaciones repetidas en E7.

## 4. Ejemplos

### 4.1 Solicitudes válidas

| Caso | `riesgo` | `dispositivoValido` | `mfaVerificado` | `estado` | ¿Cumple inv? |
|---|---|---|---|---|---|
| Riesgo bajo, todo en regla | 10 | `true` | `true` | `<PENDIENTE>` | Sí |
| Riesgo en el límite superior | 100 | `true` | `false` | `<PENDIENTE>` | Sí (100 ≤ 100) |
| Riesgo mínimo | 0 | `true` | `true` | `<PENDIENTE>` | Sí |

Una solicitud con `riesgo = 100` o `dispositivoValido = false` **es válida como registro**; que luego sea denegada lo deciden las reglas de E7 (`MotorZeroTrust`EvaluarAcceso`), no la invariante.

### 4.2 Solicitudes inválidas

| Caso | `riesgo` | Motivo | ¿Cumple inv? |
|---|---|---|---|
| Riesgo fuera de escala | 120 | 120 > 100 | **No** |
| Riesgo apenas excedido | 101 | 101 > 100 | **No** |

*(Un riesgo negativo ni siquiera es construible, porque `nat` no admite negativos.)*

### 4.3 Ejemplo ejecutable (Overture / VDM++ Toolbox)

    -- Válida: se evalúa sin error
    mk_SolicitudAcceso`SolicitudAcceso(mk_token(100), mk_token(1), mk_token(10), mk_token(50),
                                       true, 20, false, <PENDIENTE>)

    -- Inválida: "Run-Time Error 300: Type invariant was broken" por riesgo = 120
    mk_SolicitudAcceso`SolicitudAcceso(mk_token(900), mk_token(1), mk_token(10), mk_token(50),
                                       true, 120, true, <PENDIENTE>)

El segundo caso está implementado en `Pruebas`caso5()` dentro de `model/Pruebas.vpp`.

## 5. Relación con otras etapas

- **E1:** reutiliza `SolicitudId`, `UsuarioId`, `RecursoId`, `DispositivoId` y `EstadoAcceso` desde `Tipos`.
- **E5/E6:** `MotorZeroTrust` guarda las solicitudes en `historial : seq of SolicitudAcceso`SolicitudAcceso`.
- **E7:** `EvaluarAcceso` usa `usuario`, `recurso`, `dispositivoValido`, `mfaVerificado` y `riesgo`.
- **E8:** `Pruebas`caso5()` demuestra en tiempo de ejecución el fallo de la invariante con riesgo 120.