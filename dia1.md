# Qué es Openshift

Distribución de Kubernetes realizada por RedHat.
Openshift Container Platform < (Proyecto upstream) Gratuito: Openshift Origin OKD
Producto comercial de Redhat, que requiere subscripcion con Redhat (pasta).

Kubernetes + algunos añadidos.

## Kubernetes?

Orquestador de contenedores. Orquestador de gestores (docker) de contenedores.
Quién está detrás de esto? Google.

## Contenedor?

Servicio corriendo sobre la libreria de Linux en un pequeño entorno.
                                        -----               -------

Entorno aislado dentro de un SO Linux, donde ejecutamos procesos(software).
Entorno aislado... en cuanto a qué?
- Configuración de red
- Limitación de acceso al HW
- Sistema de archivos propio (programas ya instalados, configuraciones, datos...) independiente al del host.
- Variables propias de entorno

Los contenedores me permiten resolver algunos de los mismos problemas que me resuelven las maquinas virtuales, pero mejor que las MV.

INSTALAMOS Y OPERAMOS SOFTWARE

App 1    | App 2    |  App 3                    App 1 Tiene un bug... 100% CPU/RAM/IO HDD/ESPACIO HDD
------------------------------                      App1 -> Offline ... App2 ? App 3?
    Sistema Operativo                           App 1 y App2 tuvieran requisitos incompatibles
------------------------------
    Hierro

x Maquinas virtuales 

 App1    |   App 2 y App 3                      Complica la configuración / mantemiento de infra
------------------------------                  Merma de recursos y rendimiento
 SO      |        SO                            No hay un estandar definido
------------------------------
 MV 1    |        MV 2
------------------------------
    Hipervisor (VMWare, Citrix, HyperV, Oracle VirtualBox, kvm)
------------------------------
    Sistema Operativo
------------------------------
    Hierro

CONTENEDORES

 App1    |   App 2 y App 3    
------------------------------
 C 1     |        C 2
------------------------------
    Gestor de contenedores (docker, containerd, podman, CRIO) 
------------------------------
    Sistema Operativo: LINUX: (Openshift/Redhat Enterprise Linux)
------------------------------
    Hierro

Existe un estandar de lo que es un contenedor: Docker definió el estandar.
Los contenedores los creamos desde una IMAGEN DE CONTENDOR.

W3C: Regulador de estandares en el mundo WEB: HTML, CSS, XML
Open Container Initiative: Gestiona los estandares de los contenedores: Docker 2015

## IMAGEN DE CONTENEDOR

Es un triste fichero TAR (comprimido), que contiene:
  - Librerias
  - Apps
  - Ficheros de configuración
  Al descomprirmirlo vería una típica estructura de UNIX®:
      /bin        Comandos ejecutables
      /home       Usuarios
      /var        Datos que generan programas: logs... ficheros de una BBDD
      /etc        Configuraciones
      /tmp        
      /opt        Software, apps
  Lo que contiene una imagen de contenedor es un SOFTWARE YA INSTALADO POR ALGUIEN... 
                                                      CON LA CONFIGURACION QUE QUIERA ESA PERSONA!!!!
                                           NO ES UN INSTALADOR DE SOFTWARE
Además, una imagen de contenedor lleva asociados:
    - Configuración predeterminada, que puedo cambiar... algo
    - Información adicional... Para el que le interese.

Las imagenes de contenedor las podemos generar con herramientas tipo docker, podman:
    A través de un Dockerfile

Esas imágenes de contenedor, una vez generadas, que hacemos con ellas?
    Depositarlas en un regitro de imágenes de contenedor. Registry: docker hub
                                                                    Redhat: quay.io
Las empresas tienen sus propios registries de respositorios de imagenes de contenedor. Propios...
    On premisses
    Cloud

Las imágenes de contenedor se han convertido en el estandar para distribuir software.


# Podman: Pod manager
Pod
Man-> Manager

# Cloud

Conjunto de servicios (referidos al mundo TIC) que una empresa ofrece de forma automatizada a través de internet.
Tipos de servicios:
    IaaS: Infraestructura: Montón de hardware que se puede alquilar
    PaaS: Platform: BBDD, Registry
    SaaS: Software

Amazon: AWS
Microsoft: Azure
Google: GCP

# Entorno de producción
Características:
- Alta disponibilidad       - "Intentar" garantizar la continuidad del servicio, o al menos
                              cumplir con unos objetivos: 99,9%
- Escalabilidad             - Capacidad de ajustar la infra a la demanda de cada momento 
                                Hoy en dia... puede cambiar en 5 minutos... multiplicandose por 1000
                                Crecimiento / decrecimiento

Web . Entradas: 17- 19:00 salen a la venta las entradas del proximoc oncierto de Ed Sheran
                17 18:45 h .....

Cuando voy a un entorno de produccion, la forma de conseguir H/A y escalabilidad, es mediante el empleo de CLUSTERs. < Redundancia de los sistemas (HA, escalabilidad)

ENTORNO DE PROD
    Máquina 1 - HIERRO
        Gestor de contenedores: docker / podman / CRIO / ContainerD
        Contenedor: software: app 1
    Máquina 2 - HIERRO
        Gestor de contenedores: docker / podman / CRIO / ContainerD
        Contenedor: software: app 1
    Máquina 3 PUF ! - HIERRO 
        Gestor de contenedores: docker / podman / CRIO / ContainerD
        ----Contenedor: software: app 1-----
    Maquina N - HIERRO
        Gestor de contenedores: docker / podman / CRIO / ContainerD
        Contenedor: software : app 1

Commodity hardware: Hardware CUTRE : 200 maquinas i5/i7 8 CPUs 32 Gbs de RAM ...
    Si se jode una (hw), no me afecta mucho a las demas. La tiro y pincho otra.
    RaspBerryPi

Kubernetes/Openshift?
    Que gestiona Kubernetes/ Openshift.. entre otros: Docker... podman, CRIO, ContainerD

## Que añadidos aporta Openshift?
Gestion de Usuarios
RBAC? No.. esto ya viene con Kubernetes. 
Interfaz Web Muy chula
Redhat Enterprise Linux + Compilación de Kubernetes adhoc
Openshift + Operadores validados por Redhat, y garantizados
Esos operadores, algunos desarrollados por RH, otro no, amplian la funcionalidad del cluster:
    Nuevas cosas, que no hace Kubernetes:
        Alquilar bajo demanda maquinas a un cloud.
Cli nuevo para acceder a openshift, más alla del que viene con kubernetes
Un monton de utilidades orientadas a desarrolladores
    Para que los desarrolladores puedan usar un cluster de OS, sin tener idea del mundo de los contenedores.

## Recordar cómo funciona un Kubernetes / Openshift
Kubernetes/Openshift los instalamos en todos los nodos del cluster? Una parte
Kubernetes es un sistema distribuido.

A nivel del hierro en cada maquina del cluster:
- kubelet < Arranca como un servicio a nivel de SO
- En cada maquina necesitamos un gestor de contenedores: docker

Kubernetes necesita de una serie de programas: En conjunto dan el servicio de kubernetes: Plano de control: ( todos se instalan mediante pods/contenedores )
- Base de datos: etcd
- Servidor DNS: coreDNS
- Programa que determina en que máquina del cluster debe instalarse un POD: kube-scheduller <<<
- Programa que permite las comunicaciones con el cluster de kubernetes: API-server < kubectl
- Programa que controla los nodos, pods, contenedores: manager
  --------
- Driver que permite montar una red virtual / programa que controla la red Virtual: kube-proxy
    Lo necesito tener montado en cada máquina

Los nodos que alberguen los programas del plano de control, son nodos con role de master

Cliente: kubectl / oc

# Sistema distribuido
Parte1 App1 - Maquina 1
Parte2 App1 - Maquina 2
Parte3 App1 - Maquina 3
Entre todas ofrecen el servicio

# Cluster
Conjunto de maquinas dando un servicio comun a ellas
App Weblogic en cluster
App1 Clones |
App2 Clones  >    Un Balanceador de Carga
App3 Clones |

# Cluster de Kunernetes

Maquina maestra 1
    docker + kubelet
    Contenedores
        kube-proxy*
        etcd*
        apiserver*
Maquina maestra 2
    docker + kubelet
    Contenedores
        kube-proxy*
        etcd*
        apiserver*
        scheduller*
        manager*
        coreDNS*
Maquina maestra 3
    docker + kubelet
    Contenedores
        kube-proxy*
        apiserver*
        scheduller*
        manager*
        coreDNS*

Maquina trabajadora 1
    docker + kubelet
    Contenedores
        kube-proxy*
        POD B(2):
            tomcat2
            filebeat2
Maquina trabajadora 2
    docker + kubelet
    Contenedores
        kube-proxy*
        POD B(1):
            tomcat1
            filebeat1
Maquina trabajadora 3
    docker + kubelet
    Contenedores
        kube-proxy*
        POD A: 
            mysql
...
Maquina trabajadora N
    docker + kubelet
    Contenedores
        kube-proxy*

* Contenedores del plano de control de kubernetes
    Estos contenedores se ejcutan en máquinas que tengan el role master
En producción, las máquinas maestras, normalmente las dejamos para ser maestras y nada más

# POD:
Conjunto de contenedores que:
- Corren en el mismo HOST/máquina
- Comparten configuración de red < Misma IP, se llaman entre si, a través de la palabra "localhost"
- Pueden compartir volúmenes
- Escalan de la misma forma
Es la mínima unidad de trabajo en Kubernetes


Despliegue de una aplicación web JAVA dentro de un cluster de kubernetes:
    - Servidor de aplicaciones JAVA: Especie de servidor web... pero enriquecido...
        Tomcat, Weblogic, Websphere -> log de accesos a mi app
    - Base de datos: MySQL
    - Exportar en tiempo real los datos de los ficheros de log a un ElasticSearch: FluentD, FileBeat

Elasticsearch: Motor de indexación... monitorización

Esas 3 apps irían en 1 contenedor o en 2 contenedores?
    Diferentes contenedores: Escalabilidad... Escalan de forma distinta
    Tomcat y Filebeat separados <<<< Esta es la decisión que SIEMPRE TOMAMOS EN KUBERNETES
        Si quiero actualizar uno de ellos, no me afecta al otro.
        Si se cae el monitorizador... que tomcat siga... intentare arrancar otro monitorizador

Esas 3 apps irían en un POD o en 2 PODs
    2 Pods.... por lo mismo! Escalabilidad
        POD A:
            mysql
        POD B:
            tomcat
            filebeat
    Necesitamos que corran en la misma máquina? NO 

# REDES
   (172.100.0.0/16)
| red virtual, con soporte sobre la red física de empresa                                        | Red mi empresa
|-- Maquina maestra 1   ----------------------------------------------------192.168.1.1 ---------|   (192.168.1.0/24)..
|     SO: Linux (Comunicaciones de red: netFilter: 172.100.100.1 -> 172.100.0.3)                 |     254 maquina
|                                       netFilter: 172.100.100.2 -> 172.100.0.1, 172.100.0.2)    |
|                                              192.168.1.1:31000 -> tomcat.prod.es               |
|--  Maquina maestra 2   ----------------------------------------------------192.168.1.2 --------|
|     SO: Linux (Comunicaciones de red: netFilter: 172.100.100.1 -> 172.100.0.3                  |
|                                       netFilter: 172.100.100.2 -> 172.100.0.1, 172.100.0.2)    |
|                                              192.168.1.2:31000 -> tomcat.prod.es               |
|--  Maquina maestra 3   ----------------------------------------------------192.168.1.3 --------|
|     SO: Linux (Comunicaciones de red: netFilter: 172.100.100.1 -> 172.100.0.3)                 |
|                                       netFilter: 172.100.100.2 -> 172.100.0.1, 172.100.0.2)    |
|                                              192.168.1.3:31000 -> tomcat.prod.es               |
|---- coreDns:  base_de_datos.prod.es = 172.100.100.1)                                           |
|               tomcat.prod.es        = 172.100.100.2)                                           |
|--  Maquina trabajadora 1 --------------------------------------------------192.168.1.10 -------|
|     SO: Linux (Comunicaciones de red: netFilter: 172.100.100.1 -> 172.100.0.3)                 |
|                                       netFilter: 172.100.100.2 -> 172.100.0.1, 172.100.0.2)    |
|                                             192.168.1.10:31000 -> tomcat.prod.es               |
|        Contenedores                                                                            |
|-172.100.0.1 -- POD B(2):                                                                       |
|                tomcat2   --> MYSQL base_de_datos.prod.es                                       |
|                filebeat2                                                                       |
|                                                                                                |
|--  Maquina trabajadora 2 --------------------------------------------------192.168.1.11 -------|
|     SO: Linux (Comunicaciones de red: netFilter: 172.100.100.1 -> 172.100.0.3)                 |
|                                       netFilter: 172.100.100.2 -> 172.100.0.1, 172.100.0.2)    |
|                                             192.168.1.11:31000 -> tomcat.prod.es:X             |
|        Contenedores                                                                            |
|-172.100.0.2 -- POD B(1):                                                                       |
|                tomcat1   --> MYSQL base_de_datos.prod.es                                       |
|                filebeat1                                                                       |
|                                                                                                |
|--  Maquina trabajadora 3 --------------------------------------------------192.168.1.12 -------|
|     SO: Linux (Comunicaciones de red: netFilter: 172.100.100.1 -> 172.100.0.3)                 |
|                                       netFilter: 172.100.100.2 -> 172.100.0.1, 172.100.0.2)    |
|                                             192.168.1.12:31000 -> tomcat.prod.es               |
|        Contenedores                                                                            |
|-172.100.0.3 -- POD A:                                                                          |
|                mysql                                                                           |
|                                                                                                |
                                                                       Balanceador de carga  ----| 192.168.1.100 *^*
                                                                192.168.1.100:80 >               |
                                                                    http://192.168.1.1:31000     |
                                                                    http://192.168.1.2:31000     |
                                                                    http://192.168.1.3:31000     |
                                                                    http://192.168.1.10:31000    |
                                                                    http://192.168.1.11:31000    |
                                                                    http://192.168.1.12:31000    |
                                                                                                 |
                                                                        servidor DNS  -----------|
                                                                    appGuay=192.168.1.100        |
                                                                                                 |
                                                                                                 |
                                                                                PC Carlos -------|
                                                                                conectar con tomcat??? SI
                                                                                http://appGuay

Carlos escribe en su navegador: 
http://appGuay                              gracias al DNS publico (de mi empresa)
http://192.168.1.100:80                     gracias al balanceador externo al cluster
http://192.168.1.10:31000                   gracias al netFilter (que tiene una regla que ha metido kube-proxy)
nginx.prod.es:80                            coreDNS resuelve ese nombre
http://172.100.100.2:80                     Esa IP la gestiona netFilter de la maquina 192.168.1.10 de quien es?
                                                de un servicio? INGRESS CONTROLLER
http://172.100.0.1:80                       Al POD   nginx (proxy reverso)   coreDNS resuelve ese nombre
http://tomcat.prod.es:80                    Se aplica una regla de INGRESS  http://172.100.0.1:80 -> tomcat.prod.es:80 
http://172.100.100.5:80                     IP DEL SERVICIO DE TOMCAT > netFilter    
http://172.100.0.1:80                       IP de un POD de tomcat

Esta configuración no la haríamos NUNCA. Por qué?
    tomcat1   --> MYSQL 172.100.0.3
Las IPs de los PODs pueden cambiar? NO
Lo que si.. es que el POD A muera.... Se crea un nuevo POD A (quien kubernetes)... y este pod tendrá otra IP
Cómo resolvemos esto?
    Asignando un nombre de SERVICIO (resoluble a través de DNS)
    SERVICIO: base_de_datos.prod.es -> IP de un servicio de balanceo de carga: 172.100.100.1
        ^^Esta regla donde se configura? coreDNS
    En el servicio de balanceo de carga, decir, quienes son los servidores que ofrecen ese servicio: 172.100.0.3
    Balanceo de carga: 172.100.100.1 -> 172.100.0.3      Que tipo de software usamos aquí: proxy reverso:
                                                                                    nginx
                                                                                    apache httpd
                                                                                    haserver
tomcat.prod.es -> IP Balanceo: 172.100.0.2


Para que Kubernetes/OS gestionen todas esas reglas necesitamos crear en Kubernetes:
    - Un Servicio: Service
        - ClusterIP: 
            Un nombre DNS base_de_datos.prod.es -> 
                IP de balnceo de carga 172.100.100.1 -> 
                    Pod(s) que ofrece en ultima instancia el servicio 172.100.0.1 (o el que sea en cada momento)
        - NodePort:
            Igual que un clusterIP pero además abre puertos en todos los nodos del cluster que apuntan al servicio interno 
                :31000 -> tomcat.prod.es (puertos mayores que el 30000)
        - LoadBalancer:
            Igual que el nodePort, pero gestiona automaticamente el Balanceador EXTERNO *^*
            En Kubernetes...
                Puedo usar siempre servicios de tipo LoadBalancer????
                    Que necesito (un balanceador compatible con Kubernetes)
                        En un Kubernetes que monte en un cloud: El balanceador lo regala el proveedor del cloud
                        On premisses? MetalLB 
            Y en Openshift? Openshift ya viene con su balanceador.

Ingress / Egress
    Ingress.. que solo aplica a Kubernetes. No a Openshift

# Contenedores / Microservicios