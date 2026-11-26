# Etapa E1: Identificación de tipos Quote y Token

En esta etapa se definen los tipos de datos elementales sobre los cuales operará el motor de autorización (*Policy Engine*) bajo el enfoque Zero Trust en `model/Tipos.vpp`.

---

## 1. Uso de Tipos `Quote`

El tipo `Quote` en VDM++ se utiliza cuando un dato pertenece a un dominio discreto y cerrado de valores posibles. En el motor Zero Trust, estos valores no deben modelarse como cadenas de texto libres (`seq of char`), ya que representan categorías formales conocidas por las políticas de seguridad.

### Especificación en VDM++ (`model/Tipos.vpp`)

    types
      public EstadoAcceso =
        <PENDIENTE> |
        <EVALUANDO> |
        <AUTORIZADO> |
        <DENEGADO> |
        <REVOCADO>;

      public NivelRiesgo =
        <BAJO> |
        <MEDIO> |
        <ALTO> |
        <CRITICO>;

      public Rol =
        <USUARIO> |
        <ADMINISTRADOR> |
        <AUDITOR>;

### Justificación y correspondencia con la implementación
- **`EstadoAcceso`:** Representa los únicos estados válidos por los que puede transitar una solicitud o sesión dentro del ciclo de vida del motor (`<PENDIENTE>`, `<EVALUANDO>`, `<AUTORIZADO>`, `<DENEGADO>` y `<REVOCADO>`). Evita que el sistema caiga en estados indefinidos o ambiguos durante la evaluación y el *Model Checking*. *(Nota: `<EVALUANDO>` modela la fase intermedia conceptual en el autómata; en el código VDM++, `EvaluarAcceso` transiciona directamente desde `<PENDIENTE>` hacia `<AUTORIZADO>` o `<DENEGADO>` al registrar la solicitud en `historial`).*
- **`NivelRiesgo`:** Clasifica cualitativamente el riesgo del contexto en cuatro niveles cerrados (`<BAJO>`, `<MEDIO>`, `<ALTO>`, `<CRITICO>`). **Aclaración con respecto al código:** Aunque este tipo está definido en `Tipos.vpp`, tanto `SolicitudAcceso` como `MotorZeroTrust` representan y evalúan el riesgo de forma numérica (`riesgo : nat` en escala `0..100`, con umbral `riesgo < 80`), por lo que `NivelRiesgo` no se utiliza directamente en `EvaluarAcceso`.
- **`Rol`:** Restringe los perfiles de usuario a categorías estrictamente contempladas por la organización (`<USUARIO>`, `<ADMINISTRADOR>`, `<AUDITOR>`), impidiendo la asignación de roles inexistentes en la clase `Usuario`. *(Nota: Actualmente `EvaluarAcceso` no consulta directamente `GetRol()`; el control de acceso se rige por la matriz explícita `permisos`).*

---

## 2. Uso de Tipos `Token`

El tipo `Token` en VDM++ representa valores sin estructura interna expuesta cuya única operación relevante es la comparación de igualdad (`=` y `<>`). Es el mecanismo idóneo para modelar identificadores abstractos de entidades.

### Especificación en VDM++ (`model/Tipos.vpp`)

    types
      public UsuarioId     = token;
      public RecursoId     = token;
      public DispositivoId = token;
      public SolicitudId   = token;

### Justificación de cada elección
- **Abstracción de formato:** Al motor de decisiones Zero Trust no le interesa si un identificador en la implementación futura será un UUID, un entero autoincremental o un hash alfanumérico; su único requisito formal es distinguir inequívocamente cada entidad.
- **Prevención de errores de tipado:** Definir tipos separados (`UsuarioId`, `RecursoId`, `DispositivoId`, `SolicitudId`) permite distinguir conceptualmente las llaves que se usarán en los registros (`E2`), conjuntos (`E3`) y mapas (`E5`), evitando mezclar identificadores de distinta naturaleza.