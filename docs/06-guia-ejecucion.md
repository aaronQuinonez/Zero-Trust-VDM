# Guía de Ejecución y Verificación del Modelo VDM++

Esta guía explica cómo abrir el proyecto, comprobar la sintaxis y los tipos, instanciar la clase `Pruebas` y ejecutar tanto los casos funcionales como los rechazos por contrato en **Overture IDE** o **The VDM++ Toolbox**.

---

## 1. Requisitos Previos

* **Herramienta VDM++:** [Overture IDE](https://www.overturetool.org/) o **The VDM++ Toolbox v9.0.7** (`vppgde` / consola `vpp`).
* **Especificaciones del modelo:** Deben cargarse juntas las 6 clases ubicadas en `model/`:
  * `model/Tipos.vpp`
  * `model/Usuario.vpp`
  * `model/Recurso.vpp`
  * `model/SolicitudAcceso.vpp`
  * `model/MotorZeroTrust.vpp`
  * `model/Pruebas.vpp`

---

## 2. Apertura del Proyecto y Revisión de Sintaxis y Tipos

### En Overture IDE
1. Abre **Overture IDE** y selecciona `File > New > VDM++ Project` (o importa el repositorio con `File > Import > General > Existing Projects into Workspace`).
2. Verifica que los 6 archivos `.vpp` de `model/` estén incluidos en el proyecto.
3. Haz clic derecho sobre el proyecto y ejecuta **`VDM > Type Check`** (o `Ctrl + B`).
4. Comprueba en la vista **Problems** que las 6 clases pasen con **0 errores de sintaxis** y **0 errores de tipos**.

### En The VDM++ Toolbox
1. Crea un proyecto nuevo (`Project > New Project`) y agrega los 6 archivos `.vpp` de `model/` (`Project > Add File to Project`).
2. En la ventana **Manager**, selecciona todos los archivos y ejecuta:
   * **`Actions > Syntax Check`**
   * **`Actions > Type Check`**
3. Abre **`Project > Project Options > Interpreter`** y habilita las comprobaciones dinámicas (`dtc`, `inv`, `pre`, `post`).

---

## 3. Creación de Instancia y Ejecución de Pruebas

Abre el intérprete (**Debug Console** en Overture o **`Windows > Interpreter`** en VDM++ Toolbox) e inicializa la especificación:

    init
    tcov reset
    create p := new Pruebas()

### 3.1 Pruebas funcionales y de decisión (sin errores esperados)
Ejecuta las dos operaciones principales de evaluación y revocación:

    print p.run()
    print p.decisionesE8()

* **`p.run()`** devuelve `(no return value)` al superar todas las aserciones internas (`Verifica(...)`), comprobando: autorización estándar (`riesgo = 20`), denegación sin permiso, denegación por riesgo alto (`85`), denegación en recurso crítico sin MFA, autorización en recurso crítico con MFA y revocación (`true` en el primer intento, `false` en el segundo).
* **`p.decisionesE8()`** evalúa los casos de frontera y retorna la secuencia exacta:

      [ <AUTORIZADO>, <DENEGADO>, <DENEGADO>, <DENEGADO>, <DENEGADO>, <DENEGADO> ]

  correspondientes a: `riesgo = 79` (autorizado), `riesgo = 80` (denegado), `dispositivoValido = false`, usuario inactivo, usuario no registrado y recurso no registrado.

### 3.2 Pruebas de rechazo por precondición o invariante
Cada caso negativo está encapsulado en su propia operación dentro de `Pruebas` para que un error esperado no interrumpa la ejecución de los demás. Ejecútalos de forma individual:

    print p.idRepetido()
    print p.usuarioDuplicado()
    print p.recursoDuplicado()
    print p.permisoSinUsuario()
    print p.permisoSinRecurso()
    print p.caso5()

| Operación | Condición probada | Error esperado en el intérprete |
|---|---|---|
| `p.idRepetido()` | Evaluar dos veces una solicitud con el mismo `id` | `Run-Time Error 58: The pre-condition evaluated to false` (`EvaluarAcceso`) |
| `p.usuarioDuplicado()` | Registrar un `UsuarioId` ya existente | `Run-Time Error 58: The pre-condition evaluated to false` (`RegistrarUsuario`) |
| `p.recursoDuplicado()` | Registrar un `RecursoId` ya existente | `Run-Time Error 58: The pre-condition evaluated to false` (`RegistrarRecurso`) |
| `p.permisoSinUsuario()` | Conceder permiso a un `UsuarioId` no registrado | `Run-Time Error 58: The pre-condition evaluated to false` (`ConcederPermiso`) |
| `p.permisoSinRecurso()` | Conceder permiso sobre un `RecursoId` no registrado | `Run-Time Error 58: The pre-condition evaluated to false` (`ConcederPermiso`) |
| `p.caso5()` | Construir una solicitud con `riesgo = 120` | `Run-Time Error 300: Type invariant was broken` (`SolicitudAcceso`) |

*(La secuencia completa de comandos y la salida capturada se encuentran en [`entregables/E8/comandos.txt`](../entregables/E8/comandos.txt), [`entregables/E8/ejecucion.txt`](../entregables/E8/ejecucion.txt) y [`docs/05-pruebas-validacion.md`](05-pruebas-validacion.md)).*

---

## 4. Generación del Reporte de Cobertura

1. **En consola de VDM++ Toolbox** (tras ejecutar todas las operaciones de la sección anterior):

       tcov write e8_final.tc
       rtinfo e8_final.tc

   La salida reporta **100 %** de cobertura en `Pruebas`, `Recurso`, `Usuario` y `MotorZeroTrust` (contrastable con [`entregables/cobertura.E8.txt`](../entregables/cobertura.E8.txt)).
2. **En Overture IDE:** Configura una ejecución `VDM++ Application` sobre `Pruebas` con la opción *Generate coverage* activada en la pestaña **Coverage** y revisa el resultado en la vista **VDM Coverage**.