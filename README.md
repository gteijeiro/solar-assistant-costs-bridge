# SolarCost Bridge

Bridge en Python para:

- autenticarse contra SolarAssistant,
- conectarse al websocket de Phoenix LiveView de la solapa `Totales`,
- mantener un estado actualizado y persistido,
- publicar ese estado por HTTP para que otra aplicacion lo consulte.

## Estructura

- `src/sa_totals_bridge`: codigo fuente
- `requirements.txt`: dependencias
- `data/`: base SQLite local generada en ejecucion

## Instalacion

```bash
sudo apt update
sudo apt install -y python3-full python3-venv
sudo mkdir -p /opt/solarcost/bridge
sudo chown -R "$USER":"$USER" /opt/solarcost/bridge
cd /opt/solarcost/bridge
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install solarcost-bridge
sudo "$(command -v sa_bridge)" init
```

El comando recomendado es `sa_bridge`. Los comandos anteriores `sa-bridge` y `sa-totals-bridge` siguen funcionando por compatibilidad.

Si prefieres mantener el comando anterior, tambien funciona:

```bash
sudo "$(command -v sa-totals-bridge)" init
```

Si Solar Assistant corre en la misma maquina que el bridge, el asistente propone por defecto `http://127.0.0.1`.

## Desinstalacion

`pip uninstall` solo elimina el paquete instalado dentro del entorno virtual. No borra automaticamente:

- el archivo `solarcost-bridge.env`,
- la base SQLite,
- el directorio de trabajo,
- los unit files de `systemd`,
- ni los servicios habilitados.

Eso es intencional para no perder configuracion o datos sin querer.

Si quieres una desinstalacion conservando configuracion:

```bash
source .venv/bin/activate
python -m pip uninstall solarcost-bridge
```

O con el asistente interactivo:

```bash
sa_bridge uninstall
```

Si quieres tocar un servicio `system`, usa:

```bash
sudo "$(command -v sa_bridge)" uninstall
```

Si quieres una desinstalacion limpia completa:

```bash
sudo systemctl stop solarcost-bridge.service
sudo systemctl disable solarcost-bridge.service
sudo rm -f /etc/systemd/system/solarcost-bridge.service
sudo systemctl daemon-reload

rm -f /ruta/a/solarcost-bridge.env
rm -f /ruta/a/data/solar_assistant_totals.sqlite3
rm -rf /ruta/al/directorio/de/trabajo
```

## Uso diario

Para ver logs del servicio:

```bash
sudo journalctl -u solarcost-bridge.service -f
```

Para reiniciarlo:

```bash
sudo systemctl restart solarcost-bridge.service
```

Si prefieres ejecutarlo manualmente:

```bash
cd /opt/solarcost/bridge
source .venv/bin/activate
sa_bridge run
```

La API queda por defecto en `http://127.0.0.1:8765`.

## Actualizacion

```bash
cd /opt/solarcost/bridge
source .venv/bin/activate
python -m pip install --upgrade solarcost-bridge
sudo systemctl restart solarcost-bridge.service
```

- El collector hace una busqueda hacia atras con `prev-daily` y `prev-monthly`, y luego vuelve al periodo actual con `next-daily` y `next-monthly`.
- Por defecto intenta traer `12` periodos diarios anteriores y `5` periodos mensuales anteriores.
- Swagger UI esta en `/docs` y carga assets desde CDN.

## Build y publicacion

Build local:

```bash
python -m pip install --upgrade build
python -m build
```

El repo incluye workflows de GitHub Actions para:

- CI en `push` y `pull_request`,
- publicacion manual a TestPyPI con `workflow_dispatch`,
- publicacion a PyPI al crear un tag `v*`.
