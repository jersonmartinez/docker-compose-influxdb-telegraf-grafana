# Configuración de Telegraf para Monitorización

> [!NOTE]  
> Este archivo de configuración de Telegraf está diseñado para recopilar métricas del sistema en un entorno Linux y enviarlas a una base de datos InfluxDB. A continuación, se detalla la configuración y el propósito de cada bloque.

## Global Tags

```toml
[global_tags]
  customer = "DevOpsea"
  environment = "Dev"
  os = "Linux"
```

#### Descripción
Este bloque define etiquetas globales que se aplicarán a todas las métricas recopiladas por Telegraf. Las etiquetas son pares clave-valor que ayudan a identificar y clasificar las métricas.

#### Función en la Monitorización
Estas etiquetas permiten filtrar y segmentar los datos fácilmente en la base de datos de InfluxDB o en cualquier herramienta de visualización como Grafana. Por ejemplo, puedes ver todas las métricas relacionadas con el entorno de desarrollo (`environment = "Dev"`) o un cliente específico (`customer = "DevOpsea"`).

## Configuración del Agente
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
  hostname = "Ubuntu"
  omit_hostname = false
```

#### Descripción
Este bloque configura el comportamiento del agente de Telegraf, como la frecuencia de recolección de métricas y el manejo de buffers.

- **interval** = `"15s"`: Telegraf recopilará métricas cada 15 segundos.
- **round_interval** = `true`: Asegura que las recolecciones se realicen en intervalos redondeados, como al minuto exacto, mejorando la alineación de los datos.
- **metric_batch_size** = `1000`: Número máximo de métricas que se pueden enviar en una sola petición.
- **metric_buffer_limit** = `10000`: Máximo de métricas que se pueden almacenar en el buffer antes de ser descartadas si no se pueden enviar a tiempo.
- **collection_jitter** = `"0s"`: No se añade variación al intervalo de recolección.
- **flush_interval** = `"15s"`: Frecuencia con la que Telegraf enviará los datos a la base de datos configurada.
- **flush_jitter** = `"0s"`: No se añade variación al intervalo de envío.
- **precision** = `""`: Deja la precisión de las métricas a la predeterminada.
- **hostname** = `"Ubuntu"`: Define el nombre de host que se incluirá en las métricas.
- **omit_hostname** = `false`: Incluye el nombre de host en las métricas.

#### Función en la Monitorización
Este bloque define cómo y cuándo se recopilan y envían las métricas. Configuraciones como `interval`, `flush_interval`, y `metric_batch_size` impactan directamente en la eficiencia y el rendimiento de la monitorización.

## Plugins de Salida
### InfluxDB

```toml
[[outputs.influxdb]]
  urls = ["http://192.168.0.7:8086"]
  timeout = "5s"
  database = "influx"
  username = "admin"
  password = "admin"
```

#### Descripción
Este bloque define cómo y dónde Telegraf enviará las métricas. En este caso, está configurado para enviar datos a una instancia de InfluxDB.

- **urls**: URL del servidor InfluxDB.
- **timeout**: Tiempo de espera para la conexión.
- **database**: Nombre de la base de datos en InfluxDB.
- **username** y **password**: Credenciales de acceso.

#### Función en la Monitorización
Este bloque es crucial para que las métricas recopiladas por Telegraf se envíen a InfluxDB, donde se almacenarán para su análisis y visualización. La URL especifica la dirección del servidor de InfluxDB, y el nombre de la base de datos (`database = "influx"`) indica dónde se almacenarán los datos.

## Plugins de Entrada
### CPU

```toml
[[inputs.cpu]]
  percpu = true
  totalcpu = true
  collect_cpu_time = false
  report_active = false
```

#### Descripción
Recoge métricas de uso de la CPU.

- **percpu** = `true`: Recoge datos de uso de CPU por núcleo.
- **totalcpu** = `true`: Recoge datos del uso total de CPU.
- **collect_cpu_time** = `false`: No recoge el tiempo total de CPU.
- **report_active** = `false`: No informa sobre CPUs inactivas.

#### Función en la Monitorización
Permite analizar el uso de la CPU de manera detallada, identificando cuellos de botella en el rendimiento y ayudando a optimizar el uso de los recursos del sistema.

### Disco

```toml
[[inputs.disk]]
  ignore_fs = ["tmpfs", "devtmpfs", "devfs", "iso9660", "overlay", "aufs", "squashfs"]
```

#### Descripción
Recoge métricas del uso del disco, ignorando ciertos sistemas de archivos temporales y de solo lectura que no son relevantes para la mayoría de las monitorizaciones.

#### Función en la Monitorización
Monitorizar el uso del disco es esencial para detectar problemas de almacenamiento, como la falta de espacio o el uso excesivo del disco, que pueden afectar el rendimiento del sistema.

### Entrada/Salida de Disco (Disk I/O)

```toml
[[inputs.diskio]]
```

#### Descripción
Recoge métricas de entrada/salida del disco, como la cantidad de datos leídos y escritos.

#### Función en la Monitorización
Ayuda a identificar posibles cuellos de botella en el rendimiento del disco, como altas tasas de lectura/escritura que podrían ralentizar el sistema.

### Kernel

```toml
[[inputs.kernel]]
```

#### Descripción
Recoge métricas del kernel de Linux, como la carga promedio y los tiempos de interrupción.

#### Función en la Monitorización
Proporciona información sobre el rendimiento del núcleo del sistema operativo, lo que es útil para identificar problemas a nivel del sistema operativo.

### Memoria

```toml
[[inputs.mem]]
```

####  Descripción
Recoge métricas de uso de memoria.

#### Función en la Monitorización
Monitorizar la memoria es esencial para evitar problemas como la falta de memoria disponible, que puede llevar a fallos en las aplicaciones.

### Procesos

```toml
[[inputs.processes]]
```

#### Descripción
Recoge métricas sobre los procesos del sistema, como el número de procesos en ejecución, en espera, etc.

#### Función en la Monitorización
Ayuda a identificar problemas de sobrecarga de procesos, lo que puede afectar el rendimiento general del sistema.

### Swap

```toml
[[inputs.swap]]
```

#### Descripción
Recoge métricas sobre el uso de la memoria de intercambio (swap).

#### Función en la Monitorización
Monitorizar el swap es importante para evitar que el sistema se vuelva lento cuando la memoria física se agota y el sistema empieza a usar swap en disco, que es más lento.

### Sistema

```toml
[[inputs.system]]
```

#### Descripción
Recoge métricas generales del sistema, como el tiempo de actividad, carga promedio, etc.

#### Función en la Monitorización
Proporciona una visión general del estado del sistema, ayudando a detectar posibles problemas de rendimiento.

Este archivo de configuración establece una base sólida para monitorizar un sistema Linux utilizando Telegraf y enviando los datos a InfluxDB para su análisis y visualización.

---

# Monitorizar Debian desde Windows Subsystem for Linux (WSL)

> [!IMPORTANT]  
> Podemos encontrar esta configuracion en este fichero `.conf` sobre [Telegraf WSL - Debian](https://github.com/jersonmartinez/docker-compose-influxdb-telegraf-grafana/blob/main/telegraf/telegraf-wsl-linux.conf).
> 
> La configuración de Telegraf para WSL con Debian es muy similar a la que se usaba para Ubuntu, pero hay algunas diferencias clave que es importante resaltar:

### `hostname`

El nombre de host ha sido cambiado de `"Ubuntu"` a `"WSL-debian"` para reflejar que se está monitorizando un entorno de WSL (Windows Subsystem for Linux) con Debian. Esto es útil para identificar correctamente el sistema en las métricas.

```toml
hostname = "WSL-debian"
```

### `ignore_fs` en el Plugin `disk`

```toml
ignore_fs = [
  "tmpfs", "devtmpfs", "devfs", "iso9660", "overlay", "aufs", "squashfs", "none", "rootfs", "drivers", "/dev/loop0"
]
```

En la configuración para WSL con Debian, se han añadido más sistemas de archivos a la lista de ignorados:

`none` y `rootfs`: Estos son sistemas de archivos que son típicos en un entorno WSL y no suelen ser relevantes para la monitorización.
`drivers`: Específico de WSL, utilizado para gestionar los drivers.
`/dev/loop0`: Utilizado en sistemas Linux, pero en el contexto de WSL podría no ser relevante para la monitorización.

### Entorno (`environment`)

Aunque no hay un cambio directo, es importante destacar que en ambos casos, el entorno (environment) está configurado como `"Dev"`, pero podrías cambiarlo a algo como `"WSL"` para reflejar mejor el contexto en el que se está ejecutando.

### Plugins de Entrada (`inputs`)

Los mismos plugins de entrada están configurados para recolectar métricas de CPU, disco, E/S de disco, kernel, memoria, procesos, swap, y sistema en ambas configuraciones. No hay diferencias en la configuración de estos plugins entre Ubuntu y WSL con Debian.

### Consideraciones Adicionales
**WSL:** Debido a que WSL es una capa de compatibilidad para ejecutar binarios de Linux en Windows, es posible que algunos aspectos de la monitorización se comporten de manera diferente en comparación con una distribución de Linux que se ejecuta de manera nativa, especialmente en lo que respecta a la interacción con el kernel y el sistema de archivos.

> [!NOTE]  
> Las principales diferencias entre las configuraciones de Telegraf para Ubuntu y WSL con Debian son el nombre de host y la lista de sistemas de archivos que se ignoran en el plugin disk. Estas diferencias aseguran que la monitorización esté adaptada al entorno específico de WSL y que las métricas recolectadas sean precisas y relevantes.


---

# Monitorizar contenedores Docker

> [!IMPORTANT]  
> Podemos encontrar esta configuracion en este fichero `.conf` sobre [Contenedores Docker](https://github.com/jersonmartinez/docker-compose-influxdb-telegraf-grafana/blob/main/telegraf/telegraf.conf). También tienes [este segundo ejemplo](https://github.com/jersonmartinez/docker-compose-influxdb-telegraf-grafana/blob/main/telegraf/telegraf_second.conf), que se representa con otro fichero de configuración.
> 
> La configuración de Telegraf en un contenedor Docker presenta algunas diferencias importantes en comparación con las configuraciones para Ubuntu y WSL con Debian. A continuación se detallan las diferencias y se explica el propósito de cada ajuste en el contexto de un entorno Docker:

### `hostname`

El nombre de host se establece como `"docker-telegraf"` para reflejar que esta instancia de Telegraf está ejecutándose dentro de un contenedor Docker. 

```toml
hostname = "docker-telegraf"
```

> [!NOTE]  
> Esto es útil para identificar correctamente el origen de las métricas en un entorno de monitorización que puede incluir múltiples contenedores.


## `interval` y `flush_interval` en el Plugin agent

```toml
interval = "60s"
flush_interval = "10s"
```

- **interval** = `"60s"`: En la configuración para Docker, el intervalo de recolección de métricas se establece en 60 segundos, en lugar de 15 segundos. Esto puede deberse a que los contenedores suelen tener un ciclo de vida más corto o a que se prefiera un intervalo más largo para reducir la sobrecarga en entornos con muchos contenedores.

- **flush_interval** = `"10s"`: El intervalo de envío de datos a InfluxDB se reduce a 10 segundos, lo que asegura que las métricas se envíen rápidamente, a pesar del intervalo de recolección más largo.


### `urls` en el Plugin `outputs.influxdb`

```toml
urls = ["http://influxdb:8086"]
```

La URL para la base de datos InfluxDB en la configuración de Docker utiliza un nombre de servicio (`influxdb`) en lugar de una dirección IP. Esto se debe a que, dentro de un entorno Docker, los servicios suelen ser referenciados por sus nombres de servicio en lugar de por direcciones IP, facilitando la comunicación entre contenedores.

### `ignore_fs` en el Plugin `disk`

```toml
ignore_fs = ["tmpfs", "devtmpfs", "devfs", "iso9660", "overlay", "aufs", "squashfs"]
```

No hay diferencias en la configuración de `ignore_fs` para el plugin `disk` entre Ubuntu, WSL con Debian y Docker. Sin embargo, es importante señalar que en un entorno Docker, los sistemas de archivos como `overlay` y `aufs` son especialmente relevantes, ya que son comúnmente utilizados por Docker para manejar capas de contenedores.

### Otros Plugins de Entrada (`inputs`)

**Ubuntu, WSL con Debian, y Docker:** Los mismos plugins de entrada (cpu, diskio, kernel, mem, processes, swap, system) están configurados en todas las versiones. Esto asegura que las métricas esenciales del sistema sean recolectadas de manera consistente, sin importar el entorno.

> [!NOTE]  
> Aunque la configuración es la misma, el contexto en el que se ejecutan puede cambiar la interpretación de las métricas. Por ejemplo, en un entorno Docker, las métricas de CPU y memoria podrían estar limitadas al contenedor y no reflejar todo el sistema host, a menos que se configuren adecuadamente.

### Consideraciones Adicionales

> [!IMPORTANT]  
> En Docker, es común que Telegraf se ejecute en contenedores dedicados, y la configuración puede necesitar ajustes adicionales, como el mapeo de volúmenes o la configuración de permisos, para acceder a métricas del host.

> [!NOTE]  
> Las principales diferencias entre la configuración de Telegraf para Ubuntu, WSL con Debian, y Docker están en el nombre de host, los intervalos de recolección y envío de métricas, y la forma en que se direcciona la base de datos InfluxDB. Estas diferencias aseguran que la monitorización esté adaptada a las particularidades de un entorno Docker, optimizando la eficiencia y precisión de la recolección de métricas.


---

# Monitorizar Windows (10 & 11)

> [!IMPORTANT]  
> Podemos encontrar esta configuracion en este fichero `.conf` sobre [Windows](https://github.com/jersonmartinez/docker-compose-influxdb-telegraf-grafana/blob/main/telegraf/telegraf-windows.conf).
> 
> La configuración de Telegraf para un entorno Windows tiene varias diferencias importantes en comparación con las configuraciones para Ubuntu, WSL, Docker, y otros sistemas Linux. Aquí te detallo cada sección y su propósito en la monitorización:

### `hostname` y `os`

```toml
os = "Windows"
hostname = "devopsea-dev"
```

Se establece el nombre del sistema operativo como `"Windows"` y el nombre de host específico para reflejar que la monitorización se realiza en un entorno Windows.

### `agent` Configuration

> [!IMPORTANT]  
> En Windows, la ubicación del archivo de registro se ajusta para coincidir con la estructura de directorios típica de Windows, utilizando la carpeta `Program Files`.

```toml
logfile = "/Program Files/InfluxData/telegraf/telegraf.log"
```

#### Debugging y Logging

```toml
debug = false
quiet = false
```

Estas configuraciones están explícitamente definidas en Windows para controlar la verbosidad de los logs, permitiendo un control más granular en entornos donde el rendimiento y la estabilidad son críticos.

### `flush_interval` e `interval`

**Todos los entornos:**
```toml
interval = "15s"
flush_interval = "15s"
```

La frecuencia de recolección y envío de métricas se mantiene igual a 15 segundos en todos los entornos, incluido Windows. Esto asegura una consistencia en la temporalidad de las métricas recolectadas.

### `outputs.influxdb`

URL de conexión:
```toml
urls = ["http://192.168.0.7:8086"]
```

No hay cambios en la configuración del plugin de salida para InfluxDB entre los diferentes sistemas operativos. La URL, la base de datos y las credenciales permanecen constantes.

### `inputs.win_perf_counters` (Específico de Windows)

> [!NOTE]  
> En Windows, la monitorización se realiza a través de contadores de rendimiento (Performance Counters), que son características nativas del sistema operativo para medir el rendimiento del hardware y software.

> [!IMPORTANT]  
> En Windows, los contadores de rendimiento (*Performance Counters*) son mecanismos que proporcionan información sobre el rendimiento de varios aspectos del sistema operativo y las aplicaciones que se ejecutan en él. Estos contadores están organizados en objetos de rendimiento, cada uno de los cuales representa una categoría específica de métricas, como CPU, memoria, disco, red, etc.

**CPU (`Processor`):**
```toml
ObjectName = "Processor"
Counters = ["% Idle Time", "% Processor Time", etc.]
Measurement = "win_cpu"
```

Recolecta métricas detalladas sobre el uso del procesador, lo que permite monitorear la carga de trabajo y el rendimiento del CPU.

> [!NOTE]  
> **ObjectName:** `"Processor"`
> 
> **Counters:**
> 
> - `"% Idle Time"`: Tiempo en porcentaje que el procesador está inactivo.
> 
> - `"% Interrupt Time"`: Tiempo en porcentaje que el procesador está ocupado manejando interrupciones.
> 
> - `"% Privileged Time"`: Tiempo en porcentaje que el procesador pasa ejecutando código en modo privilegiado (kernel).
> 
> - `"% User Time"`: Tiempo en porcentaje que el procesador pasa ejecutando código en modo usuario (aplicaciones).
> 
> - `"% Processor Time"`: Tiempo total en porcentaje que el procesador está ocupado (suma de modo usuario y modo kernel).
> 
> - `"% DPC Time"`: Tiempo en porcentaje que el procesador pasa manejando llamadas a procedimientos diferidos (DPCs), que son una parte del manejo de interrupciones.

**Discos Lógicos (`LogicalDisk`):**
```toml
ObjectName = "LogicalDisk"
Counters = ["% Free Space", "Free Megabytes", etc.]
Measurement = "win_disk"
```

Proporciona información sobre el uso del espacio en disco y el rendimiento del disco lógico, ayudando a identificar cuellos de botella o problemas de capacidad.

**Discos Físicos (`PhysicalDisk`):**

```toml
ObjectName = "PhysicalDisk"
Counters = ["Disk Read Bytes/sec", "Disk Write Bytes/sec", etc.]
Measurement = "win_diskio"
```

Monitorea las operaciones de entrada/salida de los discos físicos, lo que es crucial para evaluar el rendimiento del almacenamiento.

**Red (`Network Interface`):**

```toml
ObjectName = "Network Interface"
Counters = ["Bytes Received/sec", "Packets Received/sec", etc.]
Measurement = "win_net"
```

Mide el tráfico de red, ayudando a monitorear la velocidad de transferencia de datos y detectar posibles problemas de conectividad.

**Sistema (`System`):**

```toml
ObjectName = "System"
Counters = ["Context Switches/sec", "Processor Queue Length", etc.]
Measurement = "win_system"
```

Recoge métricas generales del sistema, como el número de cambios de contexto por segundo, que son indicadores del rendimiento general del sistema.

**Memoria (`Memory`):**

```toml
ObjectName = "Memory"
Counters = ["Available Bytes", "Pages/sec", etc.]
Measurement = "win_mem"
```

Proporciona información sobre la disponibilidad y el uso de la memoria, lo que es fundamental para identificar problemas de memoria insuficiente o fugas de memoria.

**Archivo de Paginación (`Paging File`):**

```toml
ObjectName = "Paging File"
Counters = ["% Usage"]
Measurement = "win_swap"
```

Monitorea el uso del archivo de paginación, que es crucial en la gestión de la memoria virtual en Windows.

### Consideraciones Adicionales

- **Entorno Windows:** Al monitorear un sistema Windows, es fundamental usar los contadores de rendimiento específicos del sistema operativo para obtener métricas precisas y relevantes. A diferencia de Linux, donde se utilizan plugins más genéricos como `cpu`, `disk`, y `mem`, en Windows es necesario especificar cada objeto y contador que se desea monitorear.

- **Compatibilidad:** Asegúrate de que el agente Telegraf tenga los permisos necesarios para acceder a los contadores de rendimiento y de que el servicio de rendimiento esté habilitado en el sistema operativo Windows.

> [!NOTE]  
> La configuración de Telegraf en Windows difiere significativamente de la de Linux y otros entornos debido a la naturaleza del sistema operativo y las herramientas disponibles para la monitorización. Utiliza contadores de rendimiento nativos de Windows para recolectar métricas detalladas del sistema, lo que permite un monitoreo más preciso del rendimiento en entornos Windows.