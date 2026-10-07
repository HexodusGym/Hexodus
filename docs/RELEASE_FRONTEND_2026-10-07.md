# Frontend: cambio de contraseña en usuarios — 2026-10-07

## Procedencia y alcance

- Repositorio y destino: `HexodusGym/Hexodus:main`.
- Base: `4f80996fa064c8727940dd4ff61574c92d6f4956`.
- Rama: `codex/frontend-local-produccion-2026-10-07`.
- Fuente: carpeta local `hexodus-main`, inicialmente sin metadatos Git.

La comparación con `main`, ignorando diferencias CRLF/LF, identificó un único
archivo con cambios funcionales: `components/usuarios/usuario-modal.tsx`.
Se incorpora completo el cambio local. Los archivos de asistencia ya coinciden
funcionalmente con la base. Se conservan las notas de las entregas del 26 y 29
de septiembre y sus enlaces, ausentes en esta copia local. Se agrega esta guía
y su enlace al README. No cambian dependencias, lockfiles ni configuración.

## Comportamiento

Antes, el formulario solo mostraba los campos de contraseña al crear un usuario.
Ahora también permite establecer una nueva contraseña al editarlo:

- Ambos campos vacíos conservan la contraseña existente; el formulario entrega
  `password: undefined` y el servicio omite esa propiedad en la solicitud.
- Al capturar cualquiera de los campos se exige una contraseña de 6 a 72
  caracteres y una confirmación coincidente. Se aplican las mismas reglas al alta.
- Cada campo tiene un botón independiente para mostrar u ocultar el texto,
  con etiqueta accesible y `type="button"` para evitar envíos accidentales.
- Los campos usan `autoComplete="new-password"` y `maxLength={72}`.
- Al abrir el modal o cambiar de usuario se vacían ambos campos y se vuelve a
  ocultar su contenido. La contraseña actual no se consulta ni se muestra.

La página de usuarios y `UsuariosService.actualizarUsuario` ya propagan la
contraseña opcional mediante `PATCH /api/usuarios/:id`; no requieren cambios.
La comprobación de permisos y el procesamiento de la contraseña corresponden
al backend existente.

## Validación

| Comprobación | Resultado |
| --- | --- |
| `pnpm install --frozen-lockfile --ignore-scripts` | Correcta, sin modificar lockfiles. |
| `pnpm build` | Correcta; compilación optimizada y 18 páginas estáticas. |
| Prueba temporal del componente con hooks React simulados | 16 casos de alta/edición: vacíos, longitud mínima/máxima, confirmación distinta o incompleta y envío válido; también mostrar/ocultar y reinicio. Todos correctos. |
| `pnpm exec tsc --noEmit --incremental false` | Siete diagnósticos existentes. Comparación con el componente original de `origin/main` mediante el compilador TypeScript: mismos siete diagnósticos, sin errores nuevos. |
| `pnpm lint` | No ejecutable: el script invoca ESLint, pero falta la dependencia y su configuración. |
| `git -c core.whitespace=cr-at-eol diff --cached --check` | Sin errores de espacios. |

La instalación inicial sin `--ignore-scripts` terminó con
`ERR_PNPM_IGNORED_BUILDS` para `core-js` y `sharp` en pnpm 11.13.0. La instalación
con scripts omitidos y el build posterior terminaron correctamente. El archivo
de configuración generado por pnpm durante ese intento no forma parte del PR.
Entorno de validación: Node.js 24.18.0, Windows. Las pruebas temporales quedan
fuera de los archivos versionados y no envían solicitudes al backend.

Los siete diagnósticos de tipos aparecen en `dashboard-header.tsx`,
`compra-modal.tsx`, `socio-modal.tsx`, `detalle-usuario-modal.tsx`,
`usuarios-table.tsx`, `nueva-venta-modal.tsx` y `lib/mock-api.ts`.
La configuración existente usa `typescript.ignoreBuildErrors: true`, por lo
que el build correcto no implica que la comprobación de tipos pase.

## Revisión funcional antes del merge

No se usaron cuentas reales ni se cambiaron contraseñas en producción.
En el preview, con una cuenta de prueba autorizada:

1. Editar datos con ambos campos vacíos y comprobar que conserva su acceso.
2. Guardar una contraseña nueva y confirmar el acceso con esa contraseña.
3. Verificar que una confirmación distinta o incompleta impide guardar.
4. Crear un usuario y comprobar la validación y los controles de visibilidad.
5. Cerrar y reabrir el modal, y cambiar de usuario, verificando que los campos
   estén vacíos y ocultos.

## Publicación y reversión

Entrega mediante PR de la rama indicada hacia `main`. El merge queda a cargo
del propietario. Este trabajo no ejecuta un despliegue directo a producción;
la publicación final depende del merge y de la integración de hosting del
repositorio. No requiere migraciones ni nuevas variables de entorno.

Para revertir tras el merge, abrir un PR de reversión del commit integrado y
validar el despliegue anterior, sin reescribir el historial de `main`.
