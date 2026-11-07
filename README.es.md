# BCKMon

🇬🇧 [English](README.md) · 🇪🇸 Castellano

**Monitor de copias de seguridad.** Una PWA de solo lectura que te dice si tus copias se han hecho de verdad:
lee los correos de aviso (IMAP), comprueba archivos en un disco de red (NFS) y te avisa por push, correo,
Telegram, webhook o Microsoft Teams cuando una copia falla, falta o no ha llegado a tiempo.

BCKMon es la **edición solo de copias de [PVEMon](https://github.com/Dagarin-es/PVEMon)**: las dos se construyen
con el mismo código. BCKMon no se conecta a Proxmox ni a Docker ni escanea la red. Si además quieres vigilar
servidores Proxmox, máquinas virtuales, contenedores y discos, usa PVEMon.

> **Estable.** La versión 2.1 es la primera estable (`dagarines/bckmon:2.1.0`, también `latest`). El historial de cambios está en [`public/changelog.md`](public/changelog.md)
> (también tras el botón **ChangeLog** del menú).

## Qué incluye

| Área | Resumen |
|---|---|
| **Copias por correo** | Varias cuentas IMAP. Formato integrado de **Synology Hyper Backup**; formatos propios (Cobian, Duplicati, Veeam…) que puedes editar y duplicar. Los orígenes y las tareas se crean a partir de los correos. |
| **Comprobación de archivos (NFS)** | Mira un archivo, un patrón (`copia_*.tar`) o el archivo más reciente de una carpeta en un disco de red: tamaño y antigüedad. No lo lee ni lo copia. |
| **Tres vistas** | Ajustes › Copias: por correo, por archivo o agrupadas por origen. |
| **Alertas** | Reglas editables, límites por tarea, posponer/reactivar, aviso al saltar y al recuperarse. |
| **Notificaciones** | Push (Web Push), correo SMTP, Telegram, webhook JSON genérico y Teams (Workflows). |
| **Kiosko** | Paneles para una pantalla de pared. |
| **Usuarios** | Roles de administrador y monitor, 2FA TOTP, registro de auditoría. |
| **Idiomas** | Castellano e inglés, por usuario. |
| **Sistema** | Copia cifrada, latido (Healthchecks, Uptime Kuma…), exportar/importar la configuración de copias con PVEMon (`.bckmonx`). |

## Instalación

Imagen: [`dagarines/bckmon`](https://hub.docker.com/r/dagarines/bckmon) (amd64 y arm64). Crea una carpeta con el
`docker-compose.yml` de [`repo-bckmon/docker-compose.yml`](repo-bckmon/docker-compose.yml) (usa el puerto 3001
para poder convivir con PVEMon) y ejecuta:

```bash
echo "SECRET_KEY=$(openssl rand -base64 32)" > .env     # GUÁRDALA: cifra los secretos guardados
mkdir -p data
docker compose up -d
docker compose exec bckmon node dist/cli.js create-admin admin   # muestra la contraseña UNA vez
curl localhost:3001/api/health
```

- **Guarda `SECRET_KEY`**: cifra las contraseñas guardadas; sin ella no se pueden descifrar.
- Recuperar el acceso: `docker compose exec bckmon node dist/cli.js reset-password admin`.
- Actualizar: `docker compose pull && docker compose up -d` (las migraciones son automáticas; conserva `data/` y `.env`).
- El contenedor se ejecuta como **root** (necesario para que la comprobación NFS salga desde un puerto reservado).
- Detrás de un proxy inverso (p. ej. Nginx Proxy Manager): `http`, IP del equipo, puerto `3001`, Websockets activados; el push y la instalación como app exigen HTTPS. Cierra el puerto al resto de la red.
- Variables: tabla del [README de PVEMon](README-PVEMON.es.md#variables-de-entorno); `EDITION=bckmon` ya viene puesto en la imagen.

## PVEMon y BCKMon juntos

Puedes ejecutar los dos en el mismo equipo en puertos distintos y con **carpetas de datos separadas**. Para pasar
la configuración de copias de uno a otro, usa **Ajustes › Sistema** en cada uno: PVEMon exporta y BCKMon importa, y
al revés. El archivo va cifrado con una frase que eliges tú.

## Código e Issues

Este repositorio es un espejo del mismo código que PVEMon, publicado para que BCKMon tenga sus propios Issues y
releases. Los cambios de código se hacen en el proyecto principal y se publican en los dos.
