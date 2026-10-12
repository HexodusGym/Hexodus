# Entrega de biometría SSE — Frontend HEXODUS

Fecha: 2026-10-12
Rama: `feat/biometria-sse-produccion-2026-10-12`

## Alcance

Esta rama integra el flujo de huella con **HEXODUS BioSync** por SSE. La pantalla de asistencia mantiene una conexión al motor local (`http://localhost:4000`) y procesa eventos cuando el motor los emite. Ya no consulta el endpoint de eventos cada 750 ms.

El alta, edición y cambio de estado de un socio notifican al kiosco para que actualice su caché biométrica. El frontend valida la sesión y la autorización del dispositivo antes de enviar una asistencia al backend.

## Archivos incluidos

- rutas de sesión, eventos y callback de huella;
- pantalla de asistencia y captura de huella;
- cliente SSE y sincronización de caché del motor;
- protocolo, audio recuperable y tipos de eventos;
- pruebas automatizadas del protocolo, SSE, sincronización y audio;
- configuración de compilación TypeScript.

## Variables del frontend

Configurar en Vercel según el entorno, sin poner secretos en `NEXT_PUBLIC_*`:

```env
NEXT_PUBLIC_MOTOR_URL=http://localhost:4000
NEXT_PUBLIC_KIOSK_ID=KIOSKO-HUELLA-01
NEXT_PUBLIC_BIOMETRIC_LEGACY=false
NEXT_PUBLIC_MOTOR_TRANSPORT=sse
HUELLA_LEGACY_CLOUD_ENABLED=false
```

`NEXT_PUBLIC_API_URL` debe apuntar al backend real del cliente y conservar las variables de Supabase que ya usa el proyecto. Esta rama no incluye cambios de esquema, migraciones ni datos de Supabase.

## Validación y reversión

Antes del merge se deben ejecutar:

```bash
npm ci
npm test
npm run build
```

La rama parte de `main` del repositorio del cliente y no se fusiona automáticamente. Si la prueba de producción falla, se puede volver a `main` o revertir el commit de esta rama sin modificar la base de datos.
