# Frontend: exportación completa del historial de asistencias — 2026-09-29

## Procedencia y alcance

- Destino: `HexodusGym/Hexodus:main`.
- Base: `889d02e69efdb0edf8485c3de83a5186e560021a` (PR #21 integrado).
- Fuente: carpeta local `hexodus-main`, entregada dentro del backend, sin metadatos Git.
- Rama nueva: `codex/sincronizar-frontend-local-2026-09-29`.
- Comparación de 332 archivos locales frente a 333 archivos versionados, normalizando CRLF/LF: tres archivos de código modificados, un README distinto y una guía histórica ausente en la copia local. No hay archivos nuevos de código.

Se incorporan las tres diferencias de código completas. Se conserva la guía
`RELEASE_FRONTEND_2026-09-26.md` del repositorio y su enlace, y se agrega este
registro al README. La diferencia del README local consistía únicamente en
la ausencia de aquel enlace. Los otros 328 archivos locales ya coinciden con
la base. No cambian dependencias, lockfiles, configuración ni variables de entorno.

## Comportamiento

Antes, el componente de registros exportaba los datos cargados en el navegador,
por lo que el historial paginado podía producir un archivo limitado a una página.
Ahora, en la pestaña de historial completo, **Exportar Excel** solicita
`GET /api/asistencia/exportar` al backend, sin parámetros de paginación.

La solicitud incorpora fecha inicial, fecha final, método de acceso, estado
permitido/denegado y búsqueda sin espacios en los extremos. La página rechaza
un rango cuya fecha final sea anterior a la inicial. El botón muestra
**Generando...**, queda deshabilitado durante la descarga y publica su estado
mediante `aria-busy`. Se muestran notificaciones de éxito o error.

El servicio conserva autenticación y zona horaria, descarga la respuesta como
Blob, obtiene el nombre desde `Content-Disposition` o usa uno alternativo con
fecha, inicia la descarga y libera la URL temporal.

En el componente compartido de registros se retira el selector Excel/PDF y se
ofrece únicamente Excel. Cuando no se recibe el callback de exportación completa
(por ejemplo, en la pestaña de hoy), se conserva la exportación local de los
registros filtrados cargados. El modal de historial individual no cambia.

## Archivos

| Archivo | Cambio |
| --- | --- |
| `app/asistencia/page.tsx` | Controlador de exportación completa, validación de fechas, estado de carga y notificaciones. |
| `components/asistencia/historial-registros.tsx` | Callback con búsqueda/estado, botón Excel y estado ocupado; conserva alternativa local. |
| `lib/services/asistencia.ts` | Descarga autenticada del XLSX sin paginación; incorpora `estado` a los tipos y consulta del historial. |
| `README.md` | Enlace a esta entrega y conservación del enlace histórico. |
| `docs/RELEASE_FRONTEND_2026-09-29.md` | Alcance, comportamiento, validación y revisión pendiente. |

## Validación ejecutada

| Comprobación | Resultado |
| --- | --- |
| `pnpm install --frozen-lockfile --ignore-scripts` | Correcta, sin cambiar lockfiles. |
| `pnpm build` | Correcta: compilación optimizada y 18 páginas estáticas. |
| Prueba temporal del servicio, con fetch y DOM simulados | Correcta: filtros, autenticación, zona horaria, ausencia de paginación, nombres UTF-8/entre comillas/alternativo, rango abierto, liberación de URL y errores JSON/no JSON sin descarga. |
| `pnpm exec tsc --noEmit --incremental false` | Siete errores existentes. Comparación independiente con los tres archivos originales de `origin/main`: mismos siete diagnósticos, archivos y líneas; sin errores nuevos. |
| `pnpm lint` | No ejecutable: el script invoca ESLint, pero falta la dependencia y su configuración. |
| `git -c core.whitespace=cr-at-eol diff --cached --check` | Sin errores de espacios. |

La configuración existente usa `typescript.ignoreBuildErrors: true`; el build
no equivale a una validación de tipos correcta. Los siete errores permanecen en
dashboard, compra de inventario, formulario de socios, detalle/listado de
usuarios, nueva venta y `lib/mock-api.ts`, documentados en la entrega anterior.
Las pruebas temporales se ejecutaron fuera de los archivos versionados.

## Revisión funcional pendiente

El endpoint requerido ya está en `main` de `HexodusGym/hexodus-backend`, mediante
el [PR #12](https://github.com/HexodusGym/hexodus-backend/pull/12). Confirmar que
el entorno al que apunta `NEXT_PUBLIC_API_URL` tenga ese backend desplegado.

No se probaron sesiones reales ni descargas contra datos de producción. En el
preview, comprobar exportación con más de una página, filtros combinados,
permisos `asistencia.exportar`, rango invertido y error del servidor. Confirmar
también la descarga local de hoy y la permanencia del historial individual.

La búsqueda y el estado de la tabla siguen filtrando los registros cargados en
el cliente, mientras que la exportación completa los aplica a todo el historial
en el servidor. Por ello, la cantidad de filas del Excel puede superar la vista
actual. Este PR no cambia ese comportamiento de la tabla ni agrega paginación
al archivo. El backend genera el libro completo en memoria; el volumen máximo
y la experiencia de descarga real quedan pendientes de medir.

## Publicación y reversión

La entrega se publica como PR desde una rama nueva del repositorio de frontend.
El merge queda a cargo del propietario. No se modifica el backend ni se ejecutan
migraciones u operaciones sobre bases de datos. Si se requiere revertir después
del merge, crear y revisar un PR de reversión desde GitHub, sin force-push a `main`.
