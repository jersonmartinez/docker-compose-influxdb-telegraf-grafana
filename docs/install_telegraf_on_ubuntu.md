# Instalación de Telegraf en Ubuntu

**Actualizar el sistema**: 
Antes de instalar cualquier paquete, es una buena práctica actualizar la lista de paquetes y el sistema para asegurarse de que todas las dependencias estén actualizadas.
```bash
sudo apt update
sudo apt upgrade -y
```

**Descargar la clave GPG de InfluxData**: 
Telegraf es un producto de InfluxData. Para asegurarte de que estás descargando el software oficial, primero debes agregar la clave GPG del repositorio de InfluxData a tu sistema.
```bash
sudo apt-get install wget
```

### Paso 2: Agregar el Repositorio de InfluxData
**Descargar la clave GPG de InfluxData y verificar su integridad**:
```bash
wget -q https://repos.influxdata.com/influxdata-archive_compat.key
echo '393e8779c89ac8d958f81f942f9ad7fb82a25e133faddaf92e15b16e6ac9ce4c influxdata-archive_compat.key' | sha256sum -c && cat influxdata-archive_compat.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg > /dev/null

# Output
influxdata-archive_compat.key: OK
```

**Agregar el repositorio de Telegraf a la lista de fuentes**:
```bash
echo 'deb [signed-by=/etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg] https://repos.influxdata.com/debian stable main' | sudo tee /etc/apt/sources.list.d/influxdata.list

# Output
deb [signed-by=/etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg] https://repos.influxdata.com/debian stable main
```

### Paso 3: Instalar Telegraf
```bash
sudo apt-get update && sudo apt-get install telegraf
```

**Si hace falta el paquete gnupg, instalarlo**
```bash
sudo apt-get install gnupg
```

### Paso 4: Configuración de Telegraf

**Editar el archivo de configuración**:
- El archivo de configuración de Telegraf se encuentra en `/etc/telegraf/telegraf.conf`.
- Puedes editarlo utilizando un editor de texto como `nano` o `vim`.

```bash
sudo su
nano /etc/telegraf/telegraf.conf

# Procedemos a limpiar el fichero
echo "" > /etc/telegraf/telegraf.conf

# Introducir la nueva configuración de 
nano /etc/telegraf/telegraf.conf
```

Esta guía detalla la configuración del archivo `telegraf.conf` para la instalación de Telegraf en un sistema Linux Ubuntu, el fichero está en: `/telegraf/telegraf-ubuntu.conf`. La configuración incluye etiquetas globales, parámetros del agente, plugins de salida y entrada.

> [!NOTE]  
> Puede checar la explicación de cada segmento de la [configuración de Telegraf para Ubuntu](https://github.com/jersonmartinez/docker-compose-influxdb-telegraf-grafana/blob/main/docs/explaining_telegraf.md#configuraci%C3%B3n-de-telegraf-para-monitorizar-ubuntu).

Lo único que cambia en esta configuración respecto a la de WSL, es el `hostname`.

Esta configuración proporciona una base sólida para la recopilación de métricas del sistema utilizando Telegraf en un entorno Linux Ubuntu. Puedes personalizar aún más la configuración según tus necesidades específicas.

**Iniciar Telegraf con la nueva configuración**

```bash
telegraf --config /etc/telegraf/telegraf.conf
```

**Reiniciar Telegraf después de cambios**: 

Después de realizar cambios en la configuración, asegúrate de reiniciar el servicio Telegraf para aplicar los cambios:
```bash
sudo systemctl status telegraf
```

```bash
# Iniciar Telegraf
sudo systemctl start telegraf.service

# Detener Telegraf
sudo systemctl stop telegraf.service

# Reiniciar Telegraf
sudo systemctl restart telegraf.service
```

**Debbuging de la configuración Telegraf**
```bash
sudo telegraf --config /etc/telegraf/telegraf.conf --debug

WARN[0000]log.go:244 gosnowflake.(*defaultLogger).Warn DBUS_SESSION_BUS_ADDRESS envvar looks to be not set, this can lead to runaway dbus-daemon processes. To avoid this, set envvar DBUS_SESSION_BUS_ADDRESS=$XDG_RUNTIME_DIR/bus (if it exists) or DBUS_SESSION_BUS_ADDRESS=/dev/null. 
2024-08-09T01:32:08Z I! Loading config: /etc/telegraf/telegraf.conf
2024-08-09T01:32:08Z I! Starting Telegraf 1.31.2 brought to you by InfluxData the makers of InfluxDB
2024-08-09T01:32:08Z I! Available plugins: 234 inputs, 9 aggregators, 32 processors, 26 parsers, 60 outputs, 6 secret-stores
2024-08-09T01:32:08Z I! Loaded inputs: cpu disk diskio kernel mem processes swap system
2024-08-09T01:32:08Z I! Loaded aggregators: 
2024-08-09T01:32:08Z I! Loaded processors: 
2024-08-09T01:32:08Z I! Loaded secretstores: 
2024-08-09T01:32:08Z I! Loaded outputs: influxdb
2024-08-09T01:32:08Z I! Tags enabled: customer=DevOpsea environment=Dev host=Ubuntu os=Linux
2024-08-09T01:32:08Z I! [agent] Config: Interval:15s, Quiet:false, Hostname:"Ubuntu", Flush Interval:15s
2024-08-09T01:32:08Z D! [agent] Initializing plugins
2024-08-09T01:32:08Z D! [agent] Connecting outputs
2024-08-09T01:32:08Z D! [agent] Attempting connection to [outputs.influxdb]
2024-08-09T01:32:08Z D! [agent] Successfully connected to outputs.influxdb
2024-08-09T01:32:08Z D! [agent] Starting service inputs

2024-08-09T01:32:23Z D! [outputs.influxdb]  Wrote batch of 26 metrics in 7.029282ms
2024-08-09T01:32:23Z D! [outputs.influxdb]  Buffer fullness: 0 / 10000 metrics
```

**Monitorización de disco en Telegraf**
```bash
df -h
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           795M  1.5M  793M   1% /run
/dev/sda2        40G   11G   27G  28% /
tmpfs           3.9G     0  3.9G   0% /dev/shm
tmpfs           5.0M  8.0K  5.0M   1% /run/lock
tmpfs           795M  136K  795M   1% /run/user/1000
/dev/sr0         51M   51M     0 100% /media/ubuntu/VBox_GAs_7.0.18
```

**Monitorización de memoria RAM en Telegraf**

En este caso, a la máquina se le ha otorgado 12GB.
```
free -h
               total        used        free      shared  buff/cache   available
Mem:           7.8Gi       1.1Gi       5.8Gi        36Mi       1.1Gi       6.6Gi
Swap:          4.0Gi          0B       4.0Gi
```

**Monitorización de CPU en Telegraf**

Para ver el uso de CPU en tiempo real:
```bash
top
```
O para una vista más detallada del uso de CPU (`mpstat 1 5`):
```bash
$ mpstat 1 5
Linux 6.8.0-40-generic (ubuntu) 	08/20/2024 	_x86_64_	(3 CPU)

04:10:14 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
04:10:15 PM  all    0.34    0.00    1.72    0.00    0.00    0.69    0.00    0.00    0.00   97.24
04:10:16 PM  all    1.38    0.00    3.81    0.00    0.00    1.73    0.00    0.00    0.00   93.08
04:10:17 PM  all    1.44    0.00    5.42    0.00    0.00    0.36    0.00    0.00    0.00   92.78
04:10:18 PM  all    2.54    0.00    5.07    0.36    0.00    1.45    0.00    0.00    0.00   90.58
04:10:19 PM  all    0.70    0.00    2.09    0.00    0.00    0.70    0.00    0.00    0.00   96.52
Average:     all    1.27    0.00    3.59    0.07    0.00    0.99    0.00    0.00    0.00   94.08
```
Esto mostrará estadísticas de CPU cada segundo durante 5 segundos.

**Verificar las métricas recolectadas por Telegraf**

Para verificar que Telegraf está recolectando métricas de disco, puedes ejecutar Telegraf manualmente en modo debug:
```bash
sudo telegraf --config /etc/telegraf/telegraf.conf --test | grep disk
sudo telegraf --config /etc/telegraf/telegraf.conf --test | grep disk | grep path
```

```bash
$ sudo telegraf --config /etc/telegraf/telegraf.conf --test | grep disk | grep path

ARN[0000]log.go:244 gosnowflake.(*defaultLogger).Warn DBUS_SESSION_BUS_ADDRESS envvar looks to be not set, this can lead to runaway dbus-daemon processes. To avoid this, set envvar DBUS_SESSION_BUS_ADDRESS=$XDG_RUNTIME_DIR/bus (if it exists) or DBUS_SESSION_BUS_ADDRESS=/dev/null. 
2024-08-09T04:51:57Z I! Loading config: /etc/telegraf/telegraf.conf
2024-08-09T04:51:57Z I! Starting Telegraf 1.31.2 brought to you by InfluxData the makers of InfluxDB
2024-08-09T04:51:57Z I! Available plugins: 234 inputs, 9 aggregators, 32 processors, 26 parsers, 60 outputs, 6 secret-stores
2024-08-09T04:51:57Z I! Loaded inputs: cpu disk diskio kernel mem processes swap system
2024-08-09T04:51:57Z I! Loaded aggregators: 
2024-08-09T04:51:57Z I! Loaded processors: 
2024-08-09T04:51:57Z I! Loaded secretstores: 
2024-08-09T04:51:57Z W! Outputs are not used in testing mode!
2024-08-09T04:51:57Z I! Tags enabled: customer=DevOpsea environment=Dev host=Ubuntu os=Linux
> disk,customer=DevOpsea,device=sda2,environment=Dev,fstype=ext4,host=Ubuntu,mode=rw,os=Linux,path=/ free=28846891008u,inodes_free=2415308u,inodes_total=2621440u,inodes_used=206132u,inodes_used_percent=7.863311767578125,total=41953755136u,used=10942763008u,used_percent=27.50152842143276 1723179117000000000
> disk,customer=DevOpsea,device=sda2,environment=Dev,fstype=ext4,host=Ubuntu,mode=ro,os=Linux,path=/var/snap/firefox/common/host-hunspell free=28846891008u,inodes_free=2415308u,inodes_total=2621440u,inodes_used=206132u,inodes_used_percent=7.863311767578125,total=41953755136u,used=10942763008u,used_percent=27.50152842143276 1723179117000000000
```

Ya con esto hemos verificado que está recolectando y enviando datos a Influx. Ya solo queda hacer una buena consulta para mostrar los datos por medio de Grafana.

Monitorizar Ubuntu: 
- Si hay algún inconveniente con.

![Pasted image 20240808225341](https://github.com/user-attachments/assets/d58eef4e-d7dc-45e1-9d31-d9b1c6b1e583)

#### CPU:
Standard options -> Percent (0 - 100), Min: 0, Max: 100
Thresholds -> 80 (Red), 60 (Yellow).
```sql
SELECT mean("usage_guest") FROM "cpu" WHERE ("host" = 'Ubuntu' AND "customer" = 'DevOpsea' AND "os" = 'Linux' AND "environment" = 'Dev') AND $timeFilter GROUP BY time($__interval), "host" fill(linear)
```
![Pasted image 20240808225513](https://github.com/user-attachments/assets/9d9e5722-ee4e-40ea-9020-e6b1f1d350ac)

#### MEMORY:
Standard options -> bytes(SI), Min: 0, Max: 12000000000
Thresholds -> 2000000000 (Red).
```sql
SELECT "available" FROM "mem" WHERE ("host" = 'Ubuntu' AND "customer" = 'DevOpsea' AND "os" = 'Linux' AND "environment" = 'Dev') AND $timeFilter GROUP BY "host"
```
![Pasted image 20240808225817](https://github.com/user-attachments/assets/5d8b3073-9038-433d-8e5f-47ddd1f3ad35)

#### DISK:
Standard options -> bytes(SI), Min: 0, Max: 40000000000
Thresholds -> 5000000000 (Red).
```sql
SELECT "free" FROM "disk" WHERE ("host" = 'Ubuntu' AND "customer" = 'DevOpsea' AND "os" = 'Linux' AND "environment" = 'Dev' AND "path" = '/') AND $timeFilter GROUP BY "host"::tag
```
![Pasted image 20240808230032](https://github.com/user-attachments/assets/afb44d5d-d931-4ace-aa05-6890ef5e59e6)

### Solución de Problemas

#### Problema: El servicio Telegraf no se inicia
**Verificar los logs de Telegraf**:
```bash
$ sudo journalctl -u telegraf -f

Aug 08 19:15:45 ubuntu telegraf[3630]: 2024-08-09T01:15:45Z I! Starting Telegraf 1.31.2 brought to you by InfluxData the makers of InfluxDB
Aug 08 19:15:45 ubuntu telegraf[3630]: 2024-08-09T01:15:45Z I! Available plugins: 234 inputs, 9 aggregators, 32 processors, 26 parsers, 60 outputs, 6 secret-stores
Aug 08 19:15:45 ubuntu telegraf[3630]: 2024-08-09T01:15:45Z I! Loaded inputs: cpu disk diskio kernel mem processes swap system
Aug 08 19:15:45 ubuntu telegraf[3630]: 2024-08-09T01:15:45Z I! Loaded aggregators:
Aug 08 19:15:45 ubuntu telegraf[3630]: 2024-08-09T01:15:45Z I! Loaded processors:
Aug 08 19:15:45 ubuntu telegraf[3630]: 2024-08-09T01:15:45Z I! Loaded secretstores:
Aug 08 19:15:45 ubuntu telegraf[3630]: 2024-08-09T01:15:45Z I! Loaded outputs: influxdb
Aug 08 19:15:45 ubuntu telegraf[3630]: 2024-08-09T01:15:45Z I! Tags enabled: customer=DevOpsea environment=Dev host=Ubuntu os=Linux
Aug 08 19:15:45 ubuntu telegraf[3630]: 2024-08-09T01:15:45Z I! [agent] Config: Interval:15s, Quiet:false, Hostname:"Ubuntu", Flush Interval:15s
Aug 08 19:15:45 ubuntu systemd[1]: Started telegraf.service - Telegraf.
```

**Verificar el archivo de configuración**:
- Asegúrate de que el archivo `/etc/telegraf/telegraf.conf` no tenga errores de sintaxis.

**Probar la configuración**:
- Puedes verificar si la configuración es válida ejecutando:
```bash
telegraf --config /etc/telegraf/telegraf.conf --test
```

#### Problema: Conexión al servidor InfluxDB fallida
- **Verificar la URL del servidor InfluxDB**:
    - Asegúrate de que la URL especificada en el archivo de configuración de Telegraf sea correcta.
- **Verificar credenciales**:
    - Asegúrate de que el nombre de usuario y la contraseña sean correctos.
- **Verificar la conectividad de red**:
    - Asegúrate de que el servidor InfluxDB sea accesible desde la máquina donde se ejecuta Telegraf.

## Desinstalación de Telegraf
**Detener el servicio Telegraf**:
```bash
sudo systemctl stop telegraf
```

**Deshabilitar el servicio Telegraf**:
```bash
sudo systemctl disable telegraf
```

**Desinstalar Telegraf**:
```bash
sudo apt-get remove --purge telegraf
```

**Eliminar archivos de configuración residuales**:
```bash
sudo rm -rf /etc/telegraf
sudo rm -rf /var/lib/telegraf
```

Siguiendo estos pasos, deberías poder instalar, configurar y gestionar Telegraf en un sistema Debian o Ubuntu sin problemas. Si encuentras algún problema, revisa los logs y la configuración para identificar posibles errores y solucionarlos.
