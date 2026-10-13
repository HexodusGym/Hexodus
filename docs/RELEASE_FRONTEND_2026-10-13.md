# Preparación de entrega del frontend — 13 de octubre de 2026

## Origen y destino

- Origen del código: `JARB-s-Solutions/hexodus`, `main` en `b4196e9`.
- Destino del pull request: `HexodusGym/Hexodus`, `main` en `fe1051e` al preparar la rama.
- Rama propuesta: `release/frontend-local-produccion-2026-10-13`.

Ambos repositorios tienen historiales Git independientes. La rama parte de `HexodusGym/main` e incorpora las diferencias funcionales del frontend local. Se conservaron las notas de entregas previas que solo existen en el repositorio de producción y se evitaron cambios masivos por diferencias de saltos de línea.

## Cambios incluidos

- Permisos `biometria.sincronizar` y `biometria.enrolar` en los tipos, roles predefinidos y editor de roles. El administrador recibe ambos permisos y recepción solo el de sincronización.
- Tras eliminar un socio, la pantalla solicita la sincronización de la caché del lector; si falla, avisa al kiosco y muestra un mensaje de acción pendiente.
- Ajustes de tipos de socios, productos, usuarios y datos de prueba que permiten compilar con `ignoreBuildErrors: false`.
- Mejoras de accesibilidad en el modal de impresión de ticket.
- Plantilla `.env.biometric.example`, instrucciones del flujo biométrico SSE y README actualizado. El callback y el polling heredados quedan desactivados por defecto.
- `package-lock.json` sincronizado con las dependencias ya declaradas en `package.json`, incluidas `jspdf`, `jspdf-autotable` y `xlsx`; comando `npm test` agregado.

La entrega no modifica migraciones ni datos de Supabase. Los secretos del motor y del backend deben permanecer fuera de variables `NEXT_PUBLIC_*`.

## Configuración antes del despliegue

Configurar `NEXT_PUBLIC_API_URL` para el backend correspondiente, `NEXT_PUBLIC_MOTOR_URL` para el motor local y un identificador único en `NEXT_PUBLIC_KIOSK_ID`. Mantener `NEXT_PUBLIC_MOTOR_TRANSPORT=sse`, `NEXT_PUBLIC_BIOMETRIC_LEGACY=false` y `HUELLA_LEGACY_CLOUD_ENABLED=false`, salvo que se haya preparado explícitamente un entorno compatible con el flujo heredado. Ver [guía biométrica](BIOMETRIA-SSE-FRONT-2026-10-12.md).

## Validación local

- `npm ci`: instalación completada. El reporte de dependencias indicó **12 vulnerabilidades**: 2 bajas, 1 moderada, 8 altas y 1 crítica; requieren revisión independiente antes de aprobar producción.
- `npm test`: **19 pruebas aprobadas, 0 fallidas**.
- `npm run build`: compilación de producción y verificación de TypeScript completadas.
- `npm run lint`: no se ejecutó; el paquete `eslint` no está instalado en este repositorio.

Las pruebas usan datos y transportes simulados; no comprueban el backend real, Supabase ni el lector USB. Tras aprobar el PR, validar enrolamiento, asistencia, sincronización y permisos con el equipo de kiosco en el entorno de destino.

## Publicación y reversión

Este cambio se entrega como pull request hacia `main`, **sin merge ni despliegue directo**. Si se aprueba y despliega después, la reversión se hace desde el historial de `main` revirtiendo el commit de integración; no requiere restaurar base de datos.
