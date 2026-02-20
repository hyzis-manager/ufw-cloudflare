# UFW-Cloudflare

🌐 [English](README.md) | [Português](README.pt-BR.md) | **Español** | [中文](README.zh.md) | [Íslenska](README.is.md)

Gestiona automáticamente los [rangos de IP de Cloudflare](https://www.cloudflare.com/ips/) en UFW (Uncomplicated Firewall). Este script obtiene las direcciones IPv4 e IPv6 más recientes de Cloudflare y crea reglas de firewall para tráfico HTTP/HTTPS (puertos 80 y 443 TCP), asegurando que solo las solicitudes proxied por Cloudflare lleguen a tu servidor de origen.

Diseñado para **servidores Ubuntu** (también compatible con sistemas basados en Debian).

## ¿Por qué?

Cuando tu dominio está proxied a través de Cloudflare, todo el tráfico de los visitantes llega desde las [direcciones IP de Cloudflare](https://developers.cloudflare.com/fundamentals/concepts/cloudflare-ip-addresses/) en lugar de las IPs individuales de los visitantes. Tu firewall debe permitir estos rangos, de lo contrario el tráfico legítimo será bloqueado. Este script automatiza ese proceso y mantiene las reglas actualizadas.

## Requisitos

- **Ubuntu 20.04+** (o sistema basado en Debian)
- `ufw` instalado y habilitado
- `wget`
- Privilegios root (`sudo`)

## Inicio Rápido

```bash
wget -O ufw.sh https://raw.githubusercontent.com/hyzis-manager/ufw-cloudflare/main/ufw.sh
chmod +x ufw.sh
sudo ./ufw.sh
```

En la primera ejecución, el script:

1. Obtiene los rangos IPv4 e IPv6 actuales de Cloudflare desde `cloudflare.com/ips-v4` y `cloudflare.com/ips-v6`
2. Crea reglas de permiso en UFW para cada rango en los puertos **80** y **443** (TCP)
3. Pregunta si deseas activar **Supervision** (actualización automática diaria)

## Uso

```
sudo ./ufw.sh [opciones]
```

| Opción | Corto | Descripción |
|---|---|---|
| `--help` | `-h` | Muestra el mensaje de ayuda |
| `--purge` | `-p` | Elimina todas las reglas existentes de Cloudflare (marcadas con comentario `#cloudflare`) antes de agregar nuevas |
| `--no-new` | `-n` | No obtiene IPs ni agrega nuevas reglas (usar con `--purge` para solo eliminar reglas) |
| `--supervision` | `-s` | Activa actualización automática diaria mediante timer de systemd |

## Ejemplos

**Primera configuración** — obtener y permitir todas las IPs de Cloudflare:

```bash
sudo ./ufw.sh
```

**Actualizar reglas** — eliminar reglas antiguas y agregar las actuales:

```bash
sudo ./ufw.sh --purge
```

**Eliminar todas las reglas de Cloudflare** sin agregar nuevas:

```bash
sudo ./ufw.sh --purge --no-new
```

**Activar actualización automática diaria** (Supervision):

```bash
sudo ./ufw.sh --supervision
```

## Supervision

La función Supervision instala un **timer de systemd** que ejecuta el script una vez cada 24 horas con `--purge`, asegurando que tus reglas de firewall siempre reflejen los rangos de IP más recientes de Cloudflare.

Cuando se activa, se crean dos unidades de systemd:

- `ufw-cloudflare-supervision.service` — servicio oneshot que ejecuta el script
- `ufw-cloudflare-supervision.timer` — timer que dispara el servicio diariamente (y 5 minutos después del arranque)

### Gestión del timer

```bash
# Verificar estado del timer
systemctl status ufw-cloudflare-supervision.timer

# Ver próxima ejecución programada
systemctl list-timers ufw-cloudflare-supervision.timer

# Desactivar actualizaciones automáticas
sudo systemctl stop ufw-cloudflare-supervision.timer
sudo systemctl disable ufw-cloudflare-supervision.timer

# Disparar una actualización manualmente
sudo systemctl start ufw-cloudflare-supervision.service
```

## Cómo Funciona

1. Obtiene los rangos IPv4 de `https://www.cloudflare.com/ips-v4`
2. Obtiene los rangos IPv6 de `https://www.cloudflare.com/ips-v6`
3. Para cada rango CIDR, ejecuta: `ufw allow from <IP> to any port 80,443 proto tcp comment "cloudflare"`
4. Cuando se usa `--purge`, elimina todas las reglas UFW marcadas con el comentario `# cloudflare` antes de agregar nuevas
5. Valida los formatos de IP y verifica que la descarga fue exitosa antes de aplicar cambios

## Leyenda de Salida

Durante la ejecución, el script muestra indicadores de progreso:

- **`+`** (verde) — regla creada
- **`-`** (rojo) — regla eliminada
- **`.`** (gris) — regla omitida (ya existe o es inválida)

## Rangos de IP de Cloudflare

El script obtiene las IPs directamente de los endpoints oficiales de Cloudflare. Estos rangos también están disponibles a través de la [API de Cloudflare](https://developers.cloudflare.com/api/resources/ips/methods/list/) en `https://api.cloudflare.com/client/v4/ips` (sin necesidad de autenticación). Cloudflare actualiza estos rangos con poca frecuencia, pero recomienda mantener tu allowlist actualizada.

## Licencia

MIT
