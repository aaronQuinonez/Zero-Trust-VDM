# E9 - Model Checking de Zero Trust con NuSMV

Este directorio contiene la parte de Model Checking del proyecto
[Zero-Trust-VDM](https://github.com/aaronQuinonez/Zero-Trust-VDM).
Base revisada: commit `83a44ff002e85946614cef0be5060b5fd8c2a95e` de `main`.
La especificacion VDM++ original se conserva sin cambios.

## Archivos y alcance de esta entrega

- `model-checking/zero_trust.smv`: modelo, seis propiedades LTL y siete comprobaciones CTL.
- `model-checking/README.md`: uso e incorporacion.
- `docs/04-model-checking.md`: explicacion de E9, correspondencia con VDM++ y resultados.

El ZIP incluye solo estos tres archivos. No incluye el repositorio completo,
binarios de NuSMV, carpetas de entregables ni cambios de otros integrantes.
El documento E9 contiene la salida real de la verificacion; no hace falta un reporte separado.

## Copiar al repositorio

1. Descomprimir el ZIP en una carpeta temporal.
2. Abrir el repositorio local y comprobar `git status`. Conservar cualquier cambio previo.
3. Con el arbol limpio, actualizar `main` y crear la rama:
   ```bash
   git switch main
   git pull --ff-only origin main
   git switch -c feature/E9-model-checking-nusmv
   ```
4. Copiar la carpeta `model-checking/` a la raiz del repositorio.
5. Revisar y reemplazar `docs/04-model-checking.md` con la version del ZIP.
   Si otra persona modifico ese documento despues del commit base, comparar
   ambas versiones y conservar tambien sus aportes.
6. Ejecutar NuSMV y revisar que las 13 especificaciones terminen en `is true`.
   Un codigo de salida cero por si solo no indica que las propiedades sean verdaderas.

## Ejecutar en Windows (CMD)

Desde la raiz de Zero-Trust-VDM, usando la ruta real de NuSMV.exe:

```bat
"C:\ruta\NuSMV-2.7.1-win64\bin\NuSMV.exe" "model-checking\zero_trust.smv"
```

Sustituir la ruta de ejemplo; el ejecutable no viene incluido.
No abrir el archivo .smv en VDMTools: se ejecuta con NuSMV.

Si el ejecutable ya esta en PATH:

```bat
NuSMV.exe "model-checking\zero_trust.smv"
```

En PowerShell se necesita `&` antes de una ruta entre comillas:

```powershell
& "C:\ruta\NuSMV-2.7.1-win64\bin\NuSMV.exe" "model-checking\zero_trust.smv"
```

## Ejecutar en Linux

Con NuSMV instalado y disponible en PATH:

```bash
NuSMV model-checking/zero_trust.smv
```

Si no esta en PATH, usar la ruta completa al binario.
Version utilizada para verificar este archivo: NuSMV 2.7.1, Linux de 64 bits.

## Modo interactivo para explorar una ejecucion

```text
NuSMV -int model-checking/zero_trust.smv
go
check_fsm
pick_state -i
simulate -i -k 4
show_traces -v
quit
```

En `pick_state -i` seleccionar una configuracion inicial.
Para alcanzar AUTORIZADO, elegir datos que cumplan reglasValidas.
Para observar REVOCADO, elegir solicitarRevocacion=TRUE cuando el estado actual
sea AUTORIZADO. El cambio ocurre en el paso siguiente. Para denegar, elegir
algun dato que incumpla la regla. En Windows se usa la ruta de NuSMV.exe
en lugar de NuSMV en la primera linea.

La comprobacion batch ya verifica todas las propiedades. La simulacion
interactiva es opcional y sirve para explicar recorridos, no para sustituirla.

## Revisar y subir solo E9

```bash
git diff -- docs/04-model-checking.md
git status --short
git add model-checking/zero_trust.smv model-checking/README.md docs/04-model-checking.md
git diff --cached --stat
git commit -m "E9: modelo NuSMV y verificacion de propiedades Zero Trust"
git push -u origin feature/E9-model-checking-nusmv
```

Titulo sugerido de PR: **E9: Model Checking del ciclo de acceso Zero Trust**.

Descripcion sugerida:
> Incorpora una abstraccion finita de una solicitud con las reglas de
> MotorZeroTrust, seis propiedades LTL y siete comprobaciones CTL.
> Documenta los supuestos, la correspondencia con VDM++ y la ejecucion
> en NuSMV 2.7.1: 13 especificaciones verdaderas.
> Mantiene el codigo VDM++ existente.

## Limites

Una solicitud nueva, identificador unico implicito, datos de evaluacion fijos y
longitud del historial abstraida a 0 o 1. EVALUANDO es una fase conceptual.
No se modelan varios usuarios concurrentes, cambios de permisos durante una
sesion ni un historial arbitrariamente grande. El resultado verifica esta
abstraccion, no la infraestructura real ni automaticamente todo el codigo VDM++.

El README de la raiz queda a cargo de su responsable: marcar E9 terminado
despues de revisar y aceptar estos archivos.

