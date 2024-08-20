# Instalación de Telegraf en Debian con WSL

**Actualizar el sistema**: 
Antes de instalar cualquier paquete, es una buena práctica actualizar la lista de paquetes y el sistema para asegurarse de que todas las dependencias estén actualizadas.
```bash
sudo apt-get update
sudo apt-get upgrade
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

Esta guía detalla la configuración del archivo `telegraf.conf` para la instalación de Telegraf en un sistema Linux Debian. La configuración incluye etiquetas globales, parámetros del agente, plugins de salida y entrada.

> [!NOTE]  
> Puede checar la explicación de cada segmento de la [configuración de Telegraf para Windows Subsystem for Linux - Debian](https://github.com/jersonmartinez/docker-compose-influxdb-telegraf-grafana/blob/main/docs/explaining_telegraf.md#monitorizar-debian-desde-windows-subsystem-for-linux-wsl).

#### Configuración Global
##### [global_tags]
```toml
[global_tags]
  customer = "DevOpsea"
  environment = "Dev"
  os = "Linux"
```
- **customer**: Etiqueta personalizada para identificar al cliente, en este caso "DevOpsea".
- **environment**: Especifica el entorno, en este caso "Dev".
- **os**: Define el sistema operativo, en este caso "Linux".

#### Configuración del Agente
##### [agent]
```toml
[agent]
  interval = "15s"
  round_interval = true
  metric_batch_size = 1000
  metric_buffer_limit = 10000
  collection_jitter = "0s"
  flush_interval = "15s"
  flush_jitter = "0s"
  precision = ""
  hostname = "WSL-debian"
  omit_hostname = false
```
- **interval**: Intervalo de recopilación de métricas, establecido en 15 segundos.
- **round_interval**: Redondea los tiempos de recopilación al intervalo más cercano.
- **metric_batch_size**: Número de métricas a enviar en un solo lote.
- **metric_buffer_limit**: Cantidad máxima de métricas en el búfer.
- **collection_jitter**: Añade una variación al tiempo de recopilación para evitar picos en la carga.
- **flush_interval**: Intervalo en el que las métricas se envían al servidor, establecido en 15 segundos.
- **flush_jitter**: Añade una variación al tiempo de envío de datos.
- **precision**: Precisión de las métricas recopiladas, dejar en blanco usa la precisión predeterminada.
- **hostname**: Nombre del host reportado, en este caso "WSL-debian".
- **omit_hostname**: Si se establece en `true`, no incluye el nombre del host en las métricas.

#### Plugins de Salida
##### [[outputs.influxdb]]
```toml
[[outputs.influxdb]]
   urls = ["http://192.168.0.2:8086"]
   database = "influx"
   username = "admin"
   password = "admin"
```
- **urls**: URL del servidor InfluxDB.
- **database**: Nombre de la base de datos en InfluxDB.
- **username**: Nombre de usuario para autenticación en InfluxDB.
- **password**: Contraseña para autenticación en InfluxDB.

#### Plugins de Entrada
##### [[inputs.cpu]]
```toml
[[inputs.cpu]]
  percpu = true
  totalcpu = true
  collect_cpu_time = false
  report_active = false
```
- **percpu**: Recopila estadísticas de CPU por núcleo.
- **totalcpu**: Recopila estadísticas totales de CPU.
- **collect_cpu_time**: Recopila el tiempo de CPU en lugar de los porcentajes.
- **report_active**: Si es `true`, informa solo sobre CPU activas.

##### [[inputs.disk]]
```toml
[[inputs.disk]]
  ignore_fs = ["tmpfs", "devtmpfs", "devfs", "iso9660", "overlay", "aufs", "squashfs"]
```
- **ignore_fs**: Lista de sistemas de archivos a ignorar durante la recopilación de métricas.

##### [[inputs.diskio]]
```toml
[[inputs.diskio]]
# Plugin que recopila estadísticas de E/S de disco.
```

##### [[inputs.kernel]]
```toml
[[inputs.kernel]]
# Plugin que recopila métricas del kernel. 
```

##### [[inputs.mem]]
```toml
[[inputs.mem]]
# Plugin que recopila métricas de memoria.
```

##### [[inputs.processes]]
```toml
[[inputs.processes]]
# Plugin que recopila métricas de procesos.
```

##### [[inputs.swap]]
```toml
[[inputs.swap]]
# Plugin que recopila métricas de memoria de intercambio.
```

##### [[inputs.system]]
```toml
[[inputs.system]]
# Plugin que recopila métricas del sistema.
```

Esta configuración proporciona una base sólida para la recopilación de métricas del sistema utilizando Telegraf en un entorno Linux Debian. Puedes personalizar aún más la configuración según tus necesidades específicas.

**Iniciar Telegraf con la nueva configuración**

```bash
telegraf --config /etc/telegraf/telegraf.conf
```

**Reiniciar Telegraf después de cambios**: 
Después de realizar cambios en la configuración, asegúrate de reiniciar el servicio Telegraf para aplicar los cambios:
```bash
sudo systemctl status telegraf

System has not been booted with systemd as init system (PID 1). Can't operate.
Failed to connect to bus: Host is down
```

```bash
# Hacer el fichero ejecutable
sudo chmod +x /etc/init.d/telegraf

# Iniciar Telegraf
sudo /etc/init.d/telegraf start
Starting the process telegraf [ OK ]
telegraf process was started [ OK ]

# Detener Telegraf
sudo /etc/init.d/telegraf stop

# Reiniciar Telegraf
sudo /etc/init.d/telegraf restart
```

**Debbuging de la configuración Telegraf**
```bash
sudo telegraf --config /etc/telegraf/telegraf.conf --debug

# Si genera esta salida: 
2024-07-25T20:28:44Z D! [inputs.system]  Reading users: open /var/run/utmp: no such file or directory

# Crear el fichero (Esto ocurre en WSL) y darle permisos: 
sudo touch /var/run/utmp
sudo chmod 664 /var/run/utmp

# Con esto quedaría listo: ✅
$ sudo telegraf --config /etc/telegraf/telegraf.conf --debug
2024-07-25T20:39:28Z I! Loading config: /etc/telegraf/telegraf.conf
2024-07-25T20:39:28Z I! Starting Telegraf 1.31.2 brought to you by InfluxData the makers of InfluxDB
2024-07-25T20:39:28Z I! Available plugins: 234 inputs, 9 aggregators, 32 processors, 26 parsers, 60 outputs, 6 secret-stores
2024-07-25T20:39:28Z I! Loaded inputs: cpu disk diskio kernel mem processes swap system
2024-07-25T20:39:28Z I! Loaded aggregators:
2024-07-25T20:39:28Z I! Loaded processors:
2024-07-25T20:39:28Z I! Loaded secretstores:
2024-07-25T20:39:28Z I! Loaded outputs: influxdb
2024-07-25T20:39:28Z I! Tags enabled: customer=DevOpsea environment=Dev host=WSL-debian os=Linux
2024-07-25T20:39:28Z I! [agent] Config: Interval:15s, Quiet:false, Hostname:"WSL-debian", Flush Interval:15s
2024-07-25T20:39:28Z D! [agent] Initializing plugins
2024-07-25T20:39:28Z D! [agent] Connecting outputs
2024-07-25T20:39:28Z D! [agent] Attempting connection to [outputs.influxdb]
2024-07-25T20:39:28Z D! [agent] Successfully connected to outputs.influxdb
2024-07-25T20:39:28Z D! [agent] Starting service inputs
2024-07-25T20:39:43Z D! [outputs.influxdb]  Wrote batch of 22 metrics in 6.258321ms
2024-07-25T20:39:43Z D! [outputs.influxdb]  Buffer fullness: 0 / 10000 metrics
```

**Monitorización de disco en Telegraf**
```bash
df -h

# Lo normal es capturar /, donde está montado todo el disco: 
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdf       1007G  1.6G  955G   1% /

# En caso de WSL, también captura mis discos en Windows (/mnt/c, /mnt/e)
Filesystem      Size  Used Avail Use% Mounted on
C:\             931G  424G  508G  46% /mnt/c
E:\             477G  407G   71G  86% /mnt/e
```

**Verificar las métricas recolectadas por Telegraf**
Para verificar que Telegraf está recolectando métricas de disco, puedes ejecutar Telegraf manualmente en modo debug:
```bash
sudo telegraf --config /etc/telegraf/telegraf.conf --test | grep disk
```

```bash
$ sudo telegraf --config /etc/telegraf/telegraf.conf --test | grep disk
2024-07-25T20:45:47Z I! Loading config: /etc/telegraf/telegraf.conf
2024-07-25T20:45:47Z I! Starting Telegraf 1.31.2 brought to you by InfluxData the makers of InfluxDB
2024-07-25T20:45:47Z I! Available plugins: 234 inputs, 9 aggregators, 32 processors, 26 parsers, 60 outputs, 6 secret-stores
2024-07-25T20:45:47Z I! Loaded inputs: cpu disk diskio kernel mem processes swap system
2024-07-25T20:45:47Z I! Loaded aggregators:
2024-07-25T20:45:47Z I! Loaded processors:
2024-07-25T20:45:47Z I! Loaded secretstores:
2024-07-25T20:45:47Z W! Outputs are not used in testing mode!
2024-07-25T20:45:47Z I! Tags enabled: customer=DevOpsea environment=Dev host=WSL-debian os=Linux
> disk,customer=DevOpsea,device=sdc,environment=Dev,fstype=ext4,host=WSL-debian,mode=rw,os=Linux,path=/mnt/wsl/docker-desktop/docker-desktop-user-distro free=1026050134016u,inodes_free=67108030u,inodes_total=67108864u,inodes_used=834u,inodes_used_percent=0.0012427568435668945,total=1081101176832u,used=58687488u,used_percent=0.00571942144635107 1721940347000000000
> disk,customer=DevOpsea,device=sdf,environment=Dev,fstype=ext4,host=WSL-debian,mode=rw,os=Linux,path=/ free=1024450273280u,inodes_free=67094802u,inodes_total=67108864u,inodes_used=14062u,inodes_used_percent=0.020954012870788574,total=1081101176832u,used=1658548224u,used_percent=0.16163473008340518 1721940347000000000
> disk,customer=DevOpsea,device=sdf,environment=Dev,fstype=ext4,host=WSL-debian,mode=ro,os=Linux,path=/mnt/wslg/distro free=1024450273280u,inodes_free=67094802u,inodes_total=67108864u,inodes_used=14062u,inodes_used_percent=0.020954012870788574,total=1081101176832u,used=1658548224u,used_percent=0.16163473008340518 1721940347000000000
> disk,customer=DevOpsea,device=C:\134,environment=Dev,fstype=9p,host=WSL-debian,mode=rw,os=Linux,path=/mnt/c free=544423997440u,inodes_free=1000000u,inodes_total=999u,inodes_used=0u,inodes_used_percent=0,total=999037407232u,used=454613409792u,used_percent=45.505143901626504 1721940347000000000
> disk,customer=DevOpsea,device=E:\134,environment=Dev,fstype=9p,host=WSL-debian,mode=rw,os=Linux,path=/mnt/e free=75487477760u,inodes_free=1000000u,inodes_total=999u,inodes_used=0u,inodes_used_percent=0,total=511570866176u,used=436083388416u,used_percent=85.24398421585849 1721940347000000000
```

Ya con esto hemos verificado que está recolectando y enviando datos a Influx. Ya solo queda hacer una buena consulta para mostrar los datos por medio de Grafana.

Monitorizar Debian: 
- Si hay algún inconveniente con.

![[Pasted image 20240725155219.png]]

### Paso 5: Gestión del Servicio Telegraf

**Iniciar el servicio Telegraf**:
```bash
sudo systemctl start telegraf
```

**Habilitar Telegraf para que se inicie automáticamente al arrancar el sistema**:
```bash
sudo systemctl enable telegraf
```

**Verificar el estado del servicio**:
```bash
sudo systemctl status telegraf
```

**Reiniciar el servicio después de realizar cambios en la configuración**:
```bash
sudo systemctl restart telegraf
```

### Solución de Problemas

#### Problema: El servicio Telegraf no se inicia
**Verificar los logs de Telegraf**:
```bash
sudo journalctl -u telegraf -f
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

Siguiendo estos pasos, deberías poder instalar, configurar y gestionar Telegraf en un sistema Debian o Ubuntu sin problemas con WSL. Si encuentras algún problema, revisa los logs y la configuración para identificar posibles errores y solucionarlos.