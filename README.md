# Guía Didáctica: Instalación y Configuración de Apache Hadoop en Ubuntu 24.04

## Introducción

Apache Hadoop es un framework de código abierto diseñado para el procesamiento distribuido de grandes conjuntos de datos. Esta guía te llevará paso a paso por la instalación y configuración de Hadoop en modo pseudo-distribuido en Ubuntu 24.04.

## Componentes Principales 
### 1. HDFS (Hadoop Distributed File System) 

#### Propósito: Sistema de archivos distribuido.
Características:
- Tolerante a fallos y altamente disponible.
- Almacena datos en bloques (típicamente de 128 MB en Hadoop 2+).
- Replica los datos (por defecto, 3 copias) en diferentes nodos para evitar pérdidas.

### 2. MapReduce 

#### Propósito: Modelo de programación para procesamiento distribuido.
Fases:
- Map: Procesa los datos de entrada y genera pares clave-valor.
- Reduce: Combina los resultados del map para producir la salida final.

Ideal para: Procesamiento por lotes (batch processing), no para tiempo real.

### 3. YARN (Yet Another Resource Negotiator) 

#### Propósito: Gestiona recursos y programa tareas en el clúster.
Componentes:
- ResourceManager: Coordinador central de recursos.
- NodeManager: Gestiona recursos en cada nodo.
- ApplicationMaster: Supervisa la ejecución de cada aplicación.

### 4. Hadoop Common 

Bibliotecas y utilidades compartidas por todos los módulos.

## Ecosistema de Hadoop 

Además de sus componentes centrales, Hadoop cuenta con un rico ecosistema: 

- Hive: Permite consultar datos con un lenguaje similar a SQL.
- Pig: Lenguaje de alto nivel para flujos de datos.
- HBase: Base de datos NoSQL para acceso en tiempo real.
- Spark: Motor de procesamiento rápido en memoria (más veloz que MapReduce).
- ZooKeeper: Servicio de coordinación distribuida.
- Sqoop: Transfiere datos entre Hadoop y bases de datos relacionales.
- Flume: Recoge y agrega logs o flujos de datos.
- Oozie: Programador de flujos de trabajo.

En este tutorial, aprenderá cómo instalar y configurar Hadoop en Ubuntu.

## Requisitos Previos

- Ubuntu 24.04 LTS instalado
- Usuario con privilegios sudo
- Mínimo 4GB de RAM
- Al menos 20GB de espacio en disco
- Conexión a Internet

## Parte 0. Instalacion Ubuntu 24.04 en Windows usando WSL2

### A. Habilitar Virtualización en Windows 
Habilitar la plataforma de máquina virtual con:

```bash
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

### B. Instalar WSL2 en Windows

Verificar que Windows sea versión 2004 o superior. Se puede revisar con el comando winver.

Abrir PowerShell como administrador y habilitar WSL con:

```bash
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```
Necesario `Reiniciar el equipo`.

### C. Verificar Instalación y Activar WSL v2 de forma predeterminada
Establecer WSL2 como versión predeterminada:

```bash
wsl --set-default-version 2
```

Nota: `Si pide actualizar [descargar WSL2](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi) y ejecuta el instalador de actualización manualmente`

### D. Verificar Instalación WSL2

Para listar las instancias o distribuciones de WSL (Windows Subsystem for Linux) instaladas en Windows, se utiliza el comando en la terminal o PowerShell:

```bash
wsl --list --verbose
```
o su versión corta:
```bash
wsl -l -v
```

Este comando muestra todas las distribuciones Linux instaladas, el estado de cada una (si está corriendo o detenida) y la versión de WSL que usan. También se puede usar simplemente `wsl --list` para ver las distribuciones sin detalles adicionales.

Además, si se quiere solo listar las `distribuciones que están corriendo en windows`, se emplea:

```bash
wsl --list --running
```
Listar las distribuciones disponibles con:

```bash
wsl --list --online
```
### E. Instalar Ubuntu 24.04 en Windows usando WSL2

#### Método 1: Instalar desde la terminal con WSL

Abrir PowerShell como administrador e Instalar Ubuntu 24.04 con:

```bash
wsl --install -d Ubuntu-24.04
```
Esperar a que finalice la instalación y luego inicializar la distribución para completar la configuración inicial.

#### Método 2: Instalar desde Microsoft Store

Abrir Microsoft Store en Windows.

Buscar `Ubuntu` y elegir `Ubuntu 24.04 LTS`.

Hacer clic en `Obtener` o `Instalar`.

Una vez instalada, iniciar la aplicación para terminar la configuración.

### F. Iniciar instancia Ubuntu 24.04 en WSL2
 Abrir una ventana de PowerShell o CMD o Terminal y ejecutar el comando:

```bash
wsl -d Ubuntu-24.04
```
### G. Apagar Instancia Ubuntu 24.04 en WSL2
 Abrir una ventana de PowerShell o CMD o Terminal y ejecutar el comando:

```bash
wsl --terminate Ubuntu-24.04
```

### H. Eliminar instancia de ubuntu creada con WSL2
```bash
wsl --unregister Ubuntu-24.04
```

## Parte 1: Preparación del Sistema

### 1.1 Actualizar el Sistema

```bash
sudo apt update
sudo apt upgrade -y
```

### 1.2 Instalar Java (OpenJDK 11)

Hadoop requiere Java para funcionar. Instalaremos OpenJDK 11:

```bash
sudo apt install openjdk-11-jdk -y
```

Verificar la instalación:

```bash
java -version
```

Deberías ver algo como: `openjdk version "11.0.x"`

### 1.3 Configurar Variables de Entorno de Java

Editar el archivo de perfil:

```bash
sudo nano ~/.bashrc
```

Agregar al final del archivo:

```bash
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export PATH=$PATH:$JAVA_HOME/bin
```

Aplicar los cambios:

```bash
source ~/.bashrc
```

Verificar:

```bash
echo $JAVA_HOME
```

## Parte 2: Crear Usuario para Hadoop

### 2.1 Crear Usuario Dedicado

Es una buena práctica crear un usuario específico para Hadoop:

```bash
sudo adduser hadoop
sudo usermod -aG sudo hadoop
```

Cambiar al nuevo usuario:

```bash
su - hadoop
```

## Parte 3: Configurar SSH sin Contraseña

Hadoop requiere acceso SSH para gestionar sus nodos.

### 3.1 Instalar OpenSSH

```bash
sudo apt install openssh-server openssh-client -y
```

### 3.2 Generar Claves SSH

```bash
ssh-keygen -t rsa -P "" -f ~/.ssh/id_rsa
```

### 3.3 Configurar Autenticación sin Contraseña

```bash
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 640 ~/.ssh/authorized_keys
```

### 3.4 Verificar Conexión SSH

```bash
ssh localhost
```

La primera vez pedirá confirmar la conexión. Escribe `yes` y presiona Enter. Si todo está correcto, estarás conectado sin contraseña.

Salir de la sesión SSH:

```bash
exit
```

## Parte 4: Descargar e Instalar Hadoop

### 4.1 Descargar Hadoop

Visita el sitio oficial de Apache Hadoop o usa wget. Para Hadoop 3.4.2:

```bash
cd ~
wget https://downloads.apache.org/hadoop/common/hadoop-3.4.2/hadoop-3.4.2.tar.gz
```

### 4.2 Extraer el Archivo

```bash
tar -xzvf hadoop-3.4.2.tar.gz
sudo mv hadoop-3.4.2 /usr/local/hadoop
```

### 4.3 Configurar Permisos

```bash
sudo chown -R hadoop:hadoop /usr/local/hadoop
```

## Parte 5: Configurar Variables de Entorno de Hadoop

### 5.1 Editar .bashrc

```bash
sudo nano ~/.bashrc
```

Agregar al final:

```bash
export HADOOP_HOME=/usr/local/hadoop
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"
```

Aplicar cambios:

```bash
source ~/.bashrc
```

### 5.2 Configurar hadoop-env.sh

```bash
sudo nano $HADOOP_HOME/etc/hadoop/hadoop-env.sh
```

Buscar la línea `# export JAVA_HOME=` y reemplazarla con:

```bash
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
```

## Parte 6: Configurar Hadoop

### 6.1 Configurar core-site.xml

```bash
sudo nano $HADOOP_HOME/etc/hadoop/core-site.xml
```

Agregar entre las etiquetas `<configuration>`:

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/home/hadoop/hadoopdata/tmp</value>
    </property>
</configuration>
```

### 6.2 Configurar hdfs-site.xml

```bash
sudo nano $HADOOP_HOME/etc/hadoop/hdfs-site.xml
```

Agregar:

```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///home/hadoop/hadoopdata/hdfs/namenode</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///home/hadoop/hadoopdata/hdfs/datanode</value>
    </property>
</configuration>
```

### 6.3 Configurar mapred-site.xml

```bash
sudo nano $HADOOP_HOME/etc/hadoop/mapred-site.xml
```

Agregar:

```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.application.classpath</name>
        <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
    </property>
</configuration>
```

### 6.4 Configurar yarn-site.xml

```bash
sudo nano $HADOOP_HOME/etc/hadoop/yarn-site.xml
```

Agregar:

```xml
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_HOME,PATH,LANG,TZ,HADOOP_MAPRED_HOME</value>
    </property>
</configuration>
```

## Parte 7: Crear Directorios Necesarios

```bash
mkdir -p ~/hadoopdata/hdfs/namenode
mkdir -p ~/hadoopdata/hdfs/datanode
mkdir -p ~/hadoopdata/tmp
```

## Parte 8: Formatear el Sistema de Archivos HDFS

**Importante:** Solo se hace una vez, antes del primer inicio.

```bash
hdfs namenode -format
```

Deberías ver un mensaje indicando que el formato fue exitoso.

## Parte 9: Iniciar Hadoop

### 9.1 Iniciar HDFS

```bash
start-dfs.sh
```

### 9.2 Iniciar YARN

```bash
start-yarn.sh
```

### 9.3 Verificar Procesos en Ejecución

```bash
jps
```

Deberías ver procesos como:
- NameNode
- DataNode
- SecondaryNameNode
- ResourceManager
- NodeManager
- Jps

## Parte 10: Verificar la Instalación

### 10.1 Acceder a las Interfaces Web

- **NameNode:** http://localhost:9870
- **ResourceManager:** http://localhost:8088
- **DataNode:** http://localhost:9864

### 10.2 Comandos Básicos de HDFS

Crear un directorio:

```bash
hdfs dfs -mkdir /user
hdfs dfs -mkdir /user/hadoop
```

Listar directorios:

```bash
hdfs dfs -ls /
```

Subir un archivo:

```bash
echo "Probando Funcionamiento Apache Hadoop Local Ubuntu 24.04" > test.txt
hdfs dfs -put test.txt /user/hadoop/
```

Ver contenido de un archivo:

```bash
hdfs dfs -cat /user/hadoop/test.txt
```

Descargar un archivo:

```bash
hdfs dfs -get /user/hadoop/test.txt downloaded.txt
```

Eliminar un archivo:

```bash
hdfs dfs -rm /user/hadoop/test.txt
```

## Parte 11: Detener Hadoop

Cuando necesites detener los servicios:

```bash
stop-yarn.sh
stop-dfs.sh
```

## Solución de Problemas Comunes

### Problema 1: Error de Conexión SSH

**Solución:** Verifica que el servicio SSH esté corriendo:

```bash
sudo systemctl status ssh
sudo systemctl start ssh
```

### Problema 2: Puerto 9000 en Uso

**Solución:** Verifica qué proceso está usando el puerto:

```bash
sudo lsof -i :9000
```

### Problema 3: Permisos Denegados

**Solución:** Asegúrate de que todos los directorios de Hadoop pertenezcan al usuario hadoop:

```bash
sudo chown -R hadoop:hadoop /usr/local/hadoop
sudo chown -R hadoop:hadoop ~/hadoopdata
```

### Problema 4: Java no Encontrado

**Solución:** Verifica la variable JAVA_HOME:

```bash
echo $JAVA_HOME
ls $JAVA_HOME
```

## Ejercicios Prácticos

### Ejercicio 1: Crear una Estructura de Directorios

Crea la siguiente estructura en HDFS:

```
/proyecto
  /datos
    /entrada
    /salida
  /logs
```

### Ejercicio 2: Subir y Procesar Datos

1. Crea un archivo CSV localmente con datos de ejemplo
2. Súbelo a HDFS en `/proyecto/datos/entrada`
3. Usa comandos de HDFS para verificar su contenido

### Ejercicio 3: Monitoreo

1. Accede a la interfaz web del NameNode
2. Explora las pestañas de Datanodes, Utilities y Logs
3. Verifica el uso de espacio en HDFS

## Recursos Adicionales

- **Documentación Oficial:** https://hadoop.apache.org/docs/stable/
- **Tutoriales:** https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html
- **Comunidad:** https://hadoop.apache.org/mailing_lists.html

## Conclusión

Has instalado y configurado exitosamente Apache Hadoop en modo pseudo-distribuido en Ubuntu 24.04. Este entorno es ideal para desarrollo, pruebas y aprendizaje. Para entornos de producción, considera configurar un clúster completamente distribuido con múltiples nodos.

## Próximos Pasos

1. Aprender MapReduce y escribir tu primer programa
2. Explorar Apache Hive para consultas SQL sobre Hadoop
3. Investigar Apache Spark para procesamiento más rápido
4. Configurar un clúster multi-nodo para producción