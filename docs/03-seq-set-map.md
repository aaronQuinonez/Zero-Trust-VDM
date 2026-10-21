# Etapas E3, E5 y E6: Uso de Seq, Set, Map e Invariantes Globales

## 1. Uso de `Seq` y `Set` (E3)

En el sistema Zero Trust se utilizan ambos tipos de colección.

El tipo `Seq` se utiliza para almacenar el historial de solicitudes de acceso:

`historial : seq of SolicitudAcceso`

Se eligió una secuencia porque el orden temporal de las solicitudes es importante. De esta manera, se puede conservar el orden en que fueron realizadas.

El tipo `Set` se utiliza para representar los recursos permitidos para cada usuario:

`set of RecursoId`

Se utiliza un conjunto porque el orden de los recursos no es importante y no se requiere almacenar elementos repetidos.

---

## 2. Uso de `Map` (E5)

El sistema utiliza mapas para relacionar identificadores con las entidades principales del proyecto.

`usuarios : map UsuarioId to Usuario`

Relaciona cada identificador de usuario con su objeto `Usuario`.

`recursos : map RecursoId to Recurso`

Relaciona cada identificador de recurso con su objeto `Recurso`.

`permisos : map UsuarioId to set of RecursoId`

Relaciona cada usuario con el conjunto de recursos a los que tiene permiso de acceso.

Por ejemplo:

`U001 |-> {R001, R002}`

indica que el usuario `U001` tiene permisos sobre los recursos `R001` y `R002`.

---

## 3. Invariantes Globales (E6)

Se proponen dos invariantes para mantener la consistencia del sistema.

La primera invariante es:

`dom permisos subset dom usuarios`

Esta condición garantiza que todo usuario que tenga permisos asignados se encuentre previamente registrado en el sistema.

La segunda invariante es:

`forall uid in set dom permisos & permisos(uid) subset dom recursos`

Esta condición garantiza que todos los recursos asignados como permisos existan previamente en el mapa de recursos.

De esta manera, se evita que existan permisos asociados a usuarios o recursos inexistentes.

---

## 4. Operaciones administrativas

Para gestionar los mapas se implementan las siguientes operaciones:

- `RegistrarUsuario`
- `RegistrarRecurso`
- `ConcederPermiso`

Estas operaciones permiten registrar usuarios, registrar recursos y asignar recursos permitidos a cada usuario.

Las operaciones dinámicas de autorización y revocación se desarrollarán posteriormente en la etapa E7.