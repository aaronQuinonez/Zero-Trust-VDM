# E9: Transiciones de estado y Model Checking

## 1. Autómata de Estados (Ciclo de Vida)
El sistema Zero Trust gestiona el ciclo de vida de una solicitud de acceso mediante los siguientes estados (`EstadoAcceso`):
- `<PENDIENTE>`: La solicitud ha sido creada pero no procesada.
- `<EVALUANDO>`: El motor se encuentra comprobando las reglas R1 a R5.
- `<AUTORIZADO>`: La solicitud cumple todas las reglas y se otorga el acceso.
- `<DENEGADO>`: La solicitud falla alguna regla (sin permisos, dispositivo inválido, MFA faltante o riesgo alto).
- `<REVOCADO>`: Una sesión previamente `<AUTORIZADO>` es anulada dinámicamente.

**Transiciones válidas:**
`<PENDIENTE>` -> `<EVALUANDO>`
`<EVALUANDO>` -> `<AUTORIZADO>`
`<EVALUANDO>` -> `<DENEGADO>`
`<AUTORIZADO>` -> `<REVOCADO>`

## 2. Propiedades Críticas para Model Checking
Para la verificación formal de la fase E9, se propone comprobar la siguiente propiedad de seguridad e invariancia:

**Propiedad de Seguridad:** 
> *Ningún usuario sin permiso puede alcanzar el estado `<AUTORIZADO>`.*

**Expresión Lógica:**
`forall s in set elems historial & (s.estado = <AUTORIZADO> => (s.usuario in set dom permisos and s.recurso in set permisos(s.usuario)))`

Esta propiedad asegura que el motor nunca autorizará a un usuario si su ID no figura en la matriz de permisos o si el recurso no está explícitamente asignado a él.