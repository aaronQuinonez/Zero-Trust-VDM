# E2 – Registro `SolicitudAcceso` e invariante de riesgo

**Archivo del modelo:** `model/SolicitudAcceso.vpp`
**Depende de:** E1 (`model/Tipos.vpp`)

## 1. Propósito

Una *solicitud de acceso* es el evento que el motor Zero Trust debe evaluar: un usuario, desde un dispositivo, pide acceder a un recurso con cierto contexto (riesgo, MFA). En VDM++ se modela como un **registro compuesto** (`::`), que agrupa campos heterogéneos y permite asociarle una **invariante** que toda instancia debe cumplir.

## 2. Descripción de campos

| Campo | Tipo VDM++ | Descripción |
|---|---|---|
| `id` | `SolicitudId` (token) | Identificador único de la solicitud. |
| `usuario` | `UsuarioId` (token) | Usuario que solicita el acceso. |
| `recurso` | `RecursoId` (token) | Recurso al que se quiere acceder. |
| `dispositivo` | `DispositivoId` (token) | Dispositivo desde el que se origina la solicitud. |
| `dispositivoValido` | `bool` | `true` si el dispositivo cumple la verificación de integridad. Base de la regla **R2** en E7. |
| `riesgo` | `nat` | Puntaje de riesgo contextual de la solicitud. |
| `mfaVerificado` | `bool` | `true` si se verificó el segundo factor (necesario en recursos críticos). |
| `estado` | `EstadoAcceso` (quote) | Estado en el ciclo de vida: `<PENDIENTE>`, `<EVALUANDO>`, `<AUTORIZADO>`, `<DENEGADO>`, `<REVOCADO>`. |

## 3. Invariante de registro

```vdm
inv s == s.riesgo <= 100;
```

Formalmente: para toda solicitud `s` del tipo `SolicitudAcceso`,

$$\text{inv } s == s.\text{riesgo} \le 100$$

### Justificación

- `riesgo` es `nat`, que ya garantiza `riesgo >= 0`; solo hace falta acotar el límite superior.
- El riesgo se interpreta como una escala de 0 a 100; un valor mayor no tiene significado en el dominio.
- Al estar en el **tipo**, la invariante se comprueba cada vez que se construye o modifica una `SolicitudAcceso`: una solicitud inválida no puede existir en el modelo. Esto evita validaciones repetidas en E7.

## 4. Ejemplos

### 4.1 Solicitudes válidas

| Caso | `riesgo` | `dispositivoValido` | `mfaVerificado` | `estado` | ¿Cumple inv? |
|---|---|---|---|---|---|
| Riesgo bajo, todo en regla | 10 | `true` | `true` | `<PENDIENTE>` | Sí |
| Riesgo en el límite superior | 100 | `true` | `false` | `<PENDIENTE>` | Sí (100 ≤ 100) |
| Riesgo mínimo | 0 | `true` | `true` | `<PENDIENTE>` | Sí |

Una solicitud con `riesgo = 100` o `dispositivoValido = false` **es válida como registro**; que luego sea denegada lo deciden las reglas de E7, no la invariante.

### 4.2 Solicitudes inválidas

| Caso | `riesgo` | Motivo | ¿Cumple inv? |
|---|---|---|---|
| Riesgo fuera de escala | 120 | 120 > 100 | **No** |
| Riesgo apenas excedido | 101 | 101 > 100 | **No** |

(Un riesgo negativo ni siquiera es construible, porque `nat` no admite negativos.)

### 4.3 Ejemplo ejecutable (Overture / VDMTools)

```vdm
-- Válida: se evalúa sin error
mk_SolicitudAcceso(mk_token(1), mk_token(2), mk_token(3), mk_token(4),
                   true, 10, true, <PENDIENTE>)

-- Inválida: "Type invariant violated" por riesgo = 120
mk_SolicitudAcceso(mk_token(1), mk_token(2), mk_token(3), mk_token(4),
                   true, 120, true, <PENDIENTE>)
```

El segundo caso será el **Caso 5** de la suite de pruebas de E8.

## 5. Relación con otras etapas

- **E1:** reutiliza `SolicitudId`, `UsuarioId`, `RecursoId`, `DispositivoId` y `EstadoAcceso`.
- **E5/E6:** `MotorZeroTrust` guardará las solicitudes en `historial : seq of SolicitudAcceso`.
- **E7:** `EvaluarAcceso` usará `dispositivoValido`, `mfaVerificado` y `riesgo`.
- **E8:** el intento con riesgo 120 demostrará el fallo de la invariante.