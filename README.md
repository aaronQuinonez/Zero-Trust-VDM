# Control de acceso Zero Trust en VDM++

Proyecto del curso **Aspectos Formales de Especificación y Verificación** de la Escuela Profesional de Ingeniería de Sistemas de la Universidad Nacional de San Agustín de Arequipa (UNSA).

El [motor Zero Trust](model/MotorZeroTrust.vpp) autoriza una solicitud cuando el usuario está registrado y activo, el recurso existe, el usuario tiene permiso sobre ese recurso, el dispositivo es válido y el riesgo es menor que 80. Los recursos críticos también exigen autenticación multifactor (MFA). Si una solicitud válida incumple alguna de esas condiciones, el motor la deniega. `RevocarAcceso` cambia una autorización existente a `REVOCADO`. El rol se almacena en `Usuario`. La evaluación se basa en los permisos explícitos y el contexto de la solicitud.

## Inicio rápido

### Ejecutar las pruebas VDM++

1. Abre un proyecto VDM++ en **Overture IDE** o **The VDM++ Toolbox**, carga los seis archivos de [`model/`](model/) y comprueba la sintaxis y los tipos. Habilita las comprobaciones de tipos, invariantes, precondiciones y poscondiciones en el intérprete.
2. Inicializa el intérprete y ejecuta estas líneas. `run()` comprueba los casos base y la revocación. `decisionesE8()` devuelve una autorización para riesgo 79 y cinco denegaciones en los demás casos.

   ```text
   create p := new Pruebas()
   print p.run()
   print p.decisionesE8()
   ```

3. Para reproducir los rechazos por contrato y generar la cobertura, sigue la [guía de ejecución](docs/06-guia-ejecucion.md) y los [comandos de VDM++ Toolbox](entregables/E8/comandos.txt).

### Verificar el modelo de estados

Con **NuSMV 2.7.1** disponible en `PATH`, ejecuta este comando desde la raíz del repositorio.

```sh
NuSMV model-checking/zero_trust.smv
```

La salida esperada contiene **13 especificaciones verdaderas** (`is true`). Las [instrucciones de NuSMV](model-checking/README.md#ejecutar-en-windows-cmd) incluyen los comandos para Windows.

## Resultados y alcance

- **E8 · Pruebas y cobertura.** Los casos funcionales y los rechazos por contrato coincidieron con los resultados esperados. VDM++ Toolbox reportó **100 % de cobertura** para la ejecución registrada. El [catálogo de pruebas](docs/05-pruebas-validacion.md) detalla los escenarios, con enlaces a la [sesión](entregables/E8/ejecucion.txt) y al [informe de cobertura](entregables/cobertura.E8.txt).
- **E9 · Model checking.** NuSMV verificó **13 de 13 especificaciones** (6 LTL y 7 CTL). Las dos propiedades principales establecen que toda solicitud autorizada cumple las reglas y que toda evaluación termina en autorización o denegación. Los resultados corresponden a una abstracción de una solicitud con contexto fijo. En ella `EVALUANDO` es un estado intermedio. En VDM++ el motor registra directamente la decisión. El [informe de E9](docs/04-model-checking.md) contiene las propiedades, los supuestos y la salida de NuSMV.

## Documentación

- [Diagrama de clases](docs/clases.png) y [fuente PlantUML](docs/clases.puml).
- [Tipos Quote y Token](docs/01-quote-token.md), [registro e invariante de la solicitud](docs/02-registro-invariante.md) y [colecciones y operaciones del motor](docs/03-seq-set-map.md).
- [Referencias bibliográficas](docs/referencias.md).

## Integrantes

- Alvarez Llave, Fabian Andre
- Condori Caira, Antony Beltran
- Quiñonez Delgado, Aaron Fernando
- Riveros Valeriano, Sebastian Alfredo
- Sencia Ale, Bryan Daniel
- Yauli Merma, Diego Raul
