# OpenReply · instalación directa con Meta

Preparación para un servidor con Docker Compose v2 y un dominio apuntando
a su IP. Los puertos públicos son 80 y 443. PostgreSQL, Redis y el panel
interno no publican puertos directamente.

## Estado funcional

Esta instalación prepara OpenReply y su conexión directa con Meta. La versión
base NO inicia mensajes por recibir un nuevo seguidor. La bienvenida automática
queda pendiente de verificar acceso a un evento y permiso oficial que permitan
ese envío. No activar campañas de comentarios como sustitución de ese requisito.

## Configuración

Copiar `.env.example` a `.env` y completar las variables existentes. Añadir:

- `OPENREPLY_DOMAIN`: dominio público, sin `https://` ni ruta.
- `POSTGRES_PASSWORD`: contraseña aleatoria hexadecimal de al menos 32 bytes.
- `NEXTAUTH_URL`: `https://` seguido del mismo dominio.
- `ALLOWED_EMAILS`: los correos autorizados para entrar al panel.

Generar NEXTAUTH_SECRET, CRON_SECRET, ENCRYPTION_KEY y WEBHOOK_VERIFY_TOKEN
con un generador criptográfico. ENCRYPTION_KEY debe tener exactamente 64
caracteres hexadecimales. No subir `.env` al repositorio.

Configurar correo de acceso mediante EMAIL_SERVER o Resend y un remitente
verificado. Para Meta se necesitan INSTAGRAM_APP_ID, INSTAGRAM_APP_SECRET y
FACEBOOK_APP_SECRET. Seguir `docs/setup.md` para roles, permisos y publicación.

Los contenedores reciben automáticamente las URLs internas de base de datos y
Redis desde Compose; no es necesario publicar esos servicios.

## Arranque y comprobación

```sh
docker compose -f compose.production.yml up -d --build
docker compose -f compose.production.yml ps
docker compose -f compose.production.yml logs --tail=100 web worker cron migrate
```

Comprobar por HTTPS `/api/health`: debe devolver `status: ok` y
`checks.worker.healthy: true`. No considerar el servicio operativo hasta comprobar
también el acceso por correo y la conexión de Instagram.

En Meta, registrar bajo ese dominio `/api/instagram/callback` para OAuth y
`/api/webhook` para los eventos. No hay disparador de nuevos seguidores
configurado en esta preparación.

Compose ejecuta las migraciones antes del panel y del worker, conserva datos
en volúmenes y reinicia los procesos tras reinicios del servidor. El scheduler
mantiene las tareas de renovación de tokens. Mantener copias de PostgreSQL y
de ENCRYPTION_KEY fuera del servidor; los volúmenes no son una copia de seguridad.

## Validación realizada

Consultar el resultado de las comprobaciones en el resumen de entrega. La
validación local no acredita un despliegue público ni el envío de bienvenidas.
