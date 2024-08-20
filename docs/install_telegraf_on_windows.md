# Guía de Instalación de Telegraf en Windows

## Requisitos Previos

1. **Descargar Telegraf**:
   - Abrir el navegador y buscar en Google: `Telegraf`
   - Ve al [sitio de descargas de Telegraf](https://portal.influxdata.com/downloads/) y descarga la versión para Windows.
   - Buscar esto: https://dl.influxdata.com/telegraf/releases/telegraf-1.31.2_windows_amd64.zip para descargar de forma directa.

1. **Descomprimir el archivo descargado**:
   - Extrae el contenido del archivo ZIP descargado en una ubicación adecuada en tu sistema (por ejemplo, `C:\telegraf`).
cd <
Después, ejecutar mediante PowerShell, lo siguiente, como Administrador: 
```powershell
PS: C:\Windows\System32> cd C:\Users\jerso\Downloads

PS: C:\Users\jerso\Downloads> Expand-Archive .\telegraf-1.31.2_windows_amd64.zip -DestinationPath 'C:\Program Files\InfluxData\telegraf'
```

## Instalación de Telegraf como Servicio en Windows

### Paso 1: Configuración de Telegraf

1. **Configuración básica**:
   - Crea un archivo de configuración de Telegraf (`telegraf.conf`) o utiliza uno existente. Puedes generar un archivo de configuración predeterminado con el siguiente comando:

> Copiar el contenido de https://github.com/jersonmartinez/docker-compose-influxdb-telegraf-grafana/blob/main/telegraf/telegraf-windows.conf
sobre el fichero de configuración de telegraf.conf.

#### Descripción del fichero de configuración `telegraf.conf`
##### [global_tags]
```js
[global_tags]
  customer = "DevOpsea"
  environment = "Dev"
  os = "Windows"
```
Esta sección establece etiquetas globales que se adjuntarán a todas las métricas recolectadas. Esto es útil para identificar el origen y contexto de los datos en tus series temporales.

##### [agent]
```js
[agent]
  interval = "15s"
  round_interval = true
  metric_batch_size = 1000
  metric_buffer_limit = 10000
  collection_jitter = "0s"
  flush_interval = "15s"
  flush_jitter = "0s"
  precision = ""
  debug = false
  quiet = false
  logfile = "/Program Files/InfluxData/telegraf/telegraf.log"
  hostname = "devopsea-dev"
  omit_hostname = false
```
- **interval**: 15 segundos es un buen intervalo para la mayoría de las métricas. Si necesitas métricas más granulares, puedes reducir este valor.
- **metric_batch_size**: Un tamaño de lote de 1000 está bien para la mayoría de los casos.
- **metric_buffer_limit**: Un buffer de 10000 puede manejar picos de carga temporal, pero asegúrate de tener suficiente memoria.
- **logfile**: Asegúrate de que la ruta del archivo de log sea accesible y de que Telegraf tenga permisos para escribir en esa ubicación.
- **hostname**: `devopsea-dev` es un nombre descriptivo y ayuda a identificar el origen de los datos.

#### [[outputs.influxdb]]
```js
[[outputs.influxdb]]
   urls = ["http://192.168.0.2:8086"]
   database = "influx"
   username = "admin"
   password = "admin"
```
- **urls**: Asegúrate de que la URL del servidor InfluxDB sea accesible desde la máquina donde se ejecuta Telegraf.
- **credenciales**: Evita poner credenciales en texto plano en configuraciones de producción. Considera usar variables de entorno o un gestor de secretos.

##### [[inputs.win_perf_counters]]
```js
[[inputs.win_perf_counters]]
  [[inputs.win_perf_counters.object]]
    ObjectName = "Processor"
    Instances = ["*"]
    Counters = [
      "% Idle Time",
      "% Interrupt Time",
      "% Privileged Time",
      "% User Time",
      "% Processor Time",
      "% DPC Time",
    ]
    Measurement = "win_cpu"
    IncludeTotal=true
```
Estás capturando todas las instancias de los contadores del procesador, lo cual es muy útil para monitorear el uso de CPU. La opción `IncludeTotal=true` permite agregar una instancia total para cada contador.

##### [[inputs.win_perf_counters.object]]
```js
  [[inputs.win_perf_counters.object]]
    ObjectName = "LogicalDisk"
    Instances = ["*"]
    Counters = [
      "% Idle Time",
      "% Disk Time",
      "% Disk Read Time",
      "% Disk Write Time",
      "Current Disk Queue Length",
      "% Free Space",
      "Free Megabytes",
    ]
    Measurement = "win_disk"
```
La configuración para `LogicalDisk` captura métricas importantes de rendimiento de discos lógicos. Asegúrate de que todas las instancias necesarias sean capturadas.

##### [[inputs.win_perf_counters.object]]
```js
  [[inputs.win_perf_counters.object]]
    ObjectName = "PhysicalDisk"
    Instances = ["*"]
    Counters = [
      "Disk Read Bytes/sec",
      "Disk Write Bytes/sec",
      "Current Disk Queue Length",
      "Disk Reads/sec",
      "Disk Writes/sec",
      "% Disk Time",
      "% Disk Read Time",
      "% Disk Write Time",
    ]
    Measurement = "win_diskio"
```
Similar al bloque anterior, pero para discos físicos. Esto es útil para monitorear el rendimiento del hardware de almacenamiento.

##### [[inputs.win_perf_counters.object]]
```js
  [[inputs.win_perf_counters.object]]
    ObjectName = "Network Interface"
    Instances = ["*"]
    Counters = [
      "Bytes Received/sec",
      "Bytes Sent/sec",
      "Packets Received/sec",
      "Packets Sent/sec",
      "Packets Received Discarded",
      "Packets Outbound Discarded",
      "Packets Received Errors",
      "Packets Outbound Errors",
    ]
    Measurement = "win_net"
```
Capturar métricas de la interfaz de red te permite monitorear el tráfico y los errores de la red.

##### [[inputs.win_perf_counters.object]]
```js
  [[inputs.win_perf_counters.object]]
    ObjectName = "System"
    Counters = [
      "Context Switches/sec",
      "System Calls/sec",
      "Processor Queue Length",
      "System Up Time",
    ]
    Instances = ["------"]
    Measurement = "win_system"
```
Estas métricas del sistema son importantes para monitorear la salud general y el rendimiento del sistema operativo.

##### [[inputs.win_perf_counters.object]]
```js
  [[inputs.win_perf_counters.object]]
    ObjectName = "Memory"
    Counters = [
      "Available Bytes",
      "Cache Faults/sec",
      "Demand Zero Faults/sec",
      "Page Faults/sec",
      "Pages/sec",
      "Transition Faults/sec",
      "Pool Nonpaged Bytes",
      "Pool Paged Bytes",
      "Standby Cache Reserve Bytes",
      "Standby Cache Normal Priority Bytes",
      "Standby Cache Core Bytes",
    ]
    Instances = ["------"]
    Measurement = "win_mem"
```
Las métricas de memoria te proporcionan información sobre el uso de la memoria y la eficiencia del sistema.

##### [[inputs.win_perf_counters.object]]
```js
  [[inputs.win_perf_counters.object]]
    ObjectName = "Paging File"
    Counters = [
      "% Usage",
    ]
    Instances = ["_Total"]
    Measurement = "win_swap"
```
Monitorizar el archivo de paginación puede ayudarte a identificar problemas de memoria cuando el sistema está usando swap.

> [!NOTE]  
> Puede checar la explicación de cada segmento de la [configuración de Telegraf para Windows](https://github.com/jersonmartinez/docker-compose-influxdb-telegraf-grafana/blob/main/docs/explaining_telegraf.md#monitorizar-windows-10--11).


#### Recomendaciones Adicionales
1. **Seguridad**:
    - Considera encriptar las credenciales y usar HTTPS para las conexiones a InfluxDB.
2. **Monitoreo de Logs**:
    - Revisa regularmente el archivo de log especificado en `logfile` para identificar posibles problemas o errores en la recolección de métricas.
3. **Optimización**:
    - Ajusta los intervalos de recolección (`interval`) y de envío (`flush_interval`) según las necesidades de tu entorno para balancear la granularidad de los datos y la carga en el sistema.
Esta configuración debería proporcionarte una base sólida para monitorizar un entorno Windows utilizando Telegraf. Ajusta según las necesidades específicas de tu infraestructura y carga de trabajo.

**Iniciar el fichero de configuración:** 
```powershell
.\telegraf.exe --config "C:\Program Files\InfluxData\telegraf\telegraf.conf"
```

### Paso 2: Instalación del Servicio
1. **Instalar el servicio**:
   - Abre una terminal (PowerShell o CMD) con privilegios de administrador.
   - Navega al directorio donde descomprimiste Telegraf.
   - Ejecuta el siguiente comando para instalar Telegraf como un servicio:
```powershell
.\telegraf.exe --config "C:\Program Files\InfluxData\telegraf\telegraf.conf" --service-name telegraf service install
```

### Paso 3: Verificación del Servicio
1. **Iniciar el servicio**:
   - Después de instalar el servicio, puedes iniciarlo con el siguiente comando:

```js
net start telegraf
```

También puedes checar el servicio desde la interfaz gráfica. 
- Abre el ejecutar CTRL + R: `services.msc`

## Solución de Problemas

### Eliminar servicio

Desde la interfaz gráfica no es posible (`services.msc`), así que la eliminación es mediante línea de comandos (CMD):
```js
sc delete telegraf
```


### Problema: Clave de Registro Existente

Si al intentar instalar el servicio nuevamente, recibes un error similar al siguiente:
```js
.\telegraf.exe --config "C:\Program Files\InfluxData\telegraf\telegraf.conf" --service-name telegraf service install
```
Salida: 
```bash
setting up eventlog source failed: SYSTEM\CurrentContr01Set\Services\EventLog\App1ication\te1egraf registry key already exists
```

**Ir al ejecutar:** 
- Escribe `regedit` y busca la siguiente ruta: 
	- Ir al ejecutar y escribir `regedit` y buscar `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\EventLog\Application\telegraf`.
	- Elimina manualmente la clave de registro. 

Eliminar la clave de registro desde línea de comandos (CMD): 
```js
reg delete "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\EventLog\Application\telegraf" /f
```

### Otros Problemas Comunes 
1. **Permisos Insuficientes**:
	- Asegúrate de ejecutar todos los comandos con privilegios de administrador. 
2. **Configuración Incorrecta**: 
	- Verifica que la ruta del archivo de configuración (`telegraf.conf`) sea correcta y que el archivo esté correctamente configurado.

Siguiendo estos pasos, deberías poder instalar y ejecutar Telegraf como un servicio en Windows sin problemas. Si encuentras algún otro problema, revisa los logs de Telegraf para obtener más detalles sobre los errores y posibles soluciones.