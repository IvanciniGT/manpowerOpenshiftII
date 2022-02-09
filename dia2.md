# Kubernetes / Openshift

Se instala en cada servidor del cluster... pero 

Kubernetes/ Openshift son sistemas DISTRIBUIDOS
Distintos componentes y cada componente podía residir en un sitio  (maquina diferente)

Componentes instalados a hierro, sobre el SO de base (Linux... Openshift < Redhat):
    kubelet
        -> Gestor de contenedores: docker...

Componentes que se instalan mediante pods / contenedores: Plano de control de Kubernetes/Openshift
    coreDNS:        Servidor de DNS interno del cluster
    kube-proxy:     Alguien que gestiona las comunicaciones sobre red virtual que se va a montar entre los nodos del cluster.
                    Una por máquina
    *cni            Driver que crea y opera la red virtual a bajo nivel 
                    Una instancia en ejecución en el cluster
    api-server      Permite las comunicaciones entre todas las piezas < kubectl oc
    api-manager     Controla el trabajo del dia a dia sobre nodos, contenedores...
    etcd            BBDD
    scheduller      Determinar DONDE se instala un pod (en que maquina)
    volumenes       Almacenamiento en Red / Cloud

Creamos Pods dentro del cluster < Cierto ? Podemos? SI
                                  Lo hacemos alguna vez? NO NUNCA... Quien crea los pods es Kubernetes
                                  Vamos a dar la instrucción de crear un POD . NO

Kubernetes/OpenShift tenemos Definiciones de objetos que vamos creando dentro del cluster:
- Pod **                                            Esto representa un pod que debe crearse en el cluster
                                                    Sirve para ejecutar contenedores con SERVICIOS Y DEMONIOS
                                                    NUNCA DEFINO UN Objeto de tipo POD
                                                    Quiero se se creen y destruyan según sea necesario
Como defino PODS: Plantillas                                                    
  - Deployment                                      Plantilla de POD + número inicial de replicas
                                                    Cómo kubernetes genera esos PODs? IGUALES ENTRE SI. Clones
  - StatefulSet                                     Plantilla de POD + PLANTILLA(s) de PVC + número inicial de réplicas
                                                    Cada pod tiene su(s) propio(s) volumen(es). Esto le da personalidad
                                                    No son todos iguales
  - DaemonSet                                       Plantilla de POD
                                                    Kubernetes asegura que exista un pod de este tipo en CADA MAQUINA:
                                                            Por ejemplo: kube-proxy
- Job ~ Pod                                         NUNCA DEFINO UN Objeto de tipo JOB
                                                    Sirve para ejecutar contenedores con SCRIPTS
                                                    Quiero se se creen y destruyan según sea necesario
Como defino JOBS: Con plantillas
    - CronJob                                       PLANTILLA DE JOB y cuando arrancarlo (CRON)

- PersistentVolume
- PersistentVolumeClaim (PVC)
- Service                                           Que es? 
                                                      - ClusterIP:      Entrada en el DNS de Kubernetes
                                                                        Balanceo de carga entre PODs (reglas de netFilter)
                                                                        EXPONER PUERTOS DE PODs PARA COMUNICACIONES INTERNAS
                                                                        Estimación: Todos menos 1
                                                      - NodePort:       = ClusterIP
                                                                        + Otra regla en netFilter adicional (NAT) > 30000
                                                                        EXPONER PUERTOS DE PODs PARA COMUNICACIONES EXTERNAS 
                                                                        Estimación: 0
                                                      - LoadBalancer    = NodePort
                                                                        + Configuración automática de un Balanceador Externo:
                                                                            Entre que balancea el balanceador externo?
                                                                                Balancea petiones externas al cluster
                                                                                entre los nodos del cluster
                                                                        EXPONER PUERTOS DE PODs PARA COMUNICACIONES EXTERNAS
                                                                            CON GESTION AUTOMATICA DE UN BALANCEADOR EXTERNO
                                                                        Estimación: 1 > INGRESS_CONTROLLER / PROXY REVERSO
                                                    Sirve para publicar lo que ofrece un POD (o varios en cluster) a través de un puerto al resto del cluster o incluso al exterior del cluster
- ConfigMap
- Secret

- Namespace / Project                               Dentro de un cluster deino varios namespaces... Pero que es?
                                                    Agrupación lógica de objetos de kubernetes.
                                                    Al crear un objeto de kubernetes le enchufo a un namespace.
                                                    Espacio de nombres... 
                                                        En un namespace no puede haber 2 objetos con el mismo nombre
                                                    PARA QUE SIRVE?
                                                        En kubernetes y OS a un namespace le puedo:
                                                            - Limitar los recursos
                                                            - Dar reglas diferentes de permisos (usuario, red)
    appFacturas en cluster de kubernetes 2 veces? 
        En namespaces distintos: desarrollo, produccion
                                 cliente 1, cliente 2
                                 dpto 1, dpto 2


- Ingress / No existe en Openshift (o si....)       Proxy Reverso / Gestión de colas: IngressController
                                                    No hace balanceo de carga. Quien lo hace? netFilter en cada maquina
                                                    INGRESS CONTROLLER es un nginx, por ejemplo que se instala y corre dentro de un pod/contenedor en el cluster. Yo elijo en mi cluster cúal instalo.
                                                    INGRESS: Reglas para alimentar al INGRESS CONTROLLER
- Router ~ Ingress
- LimitRange
- ResourceQuota
- NetworkPolicy

ficheros.YAML -> Kubernetes/OS

HELM ?                                              Esto es un software externo a kubernetes. Yo puedo usarlo o no.
                                                        Escribe por mi el fichero YAML y lo carga en kubernetes.
KUSTOMIZE                                           Esto es un software externo a kubernetes. Yo puedo usarlo o no.*
                                                        Escribe por mi el fichero YAML



# COMUNICACIONES EN EL CLUSTER:

Un cliente INTERNO del cluster (proceso) que se quiere conectar con un puerto de alguien (contenedor)? MYSQL
1º El proceso habla con > coreDNS (nombreServicio) > IP de balanceo (IP Servicio)
    -- Quien hay detrás de esa IP? NADIE ... reglas en netFilter
2º El proceso hace una peticion a la IP de BALANCEO < Interceptada por el netFilter de QUE MAQUINA? De la de origen
3º Que hace netfilter? La pasa a un POD de destino

Un cliente EXTERNO al cluster (navegador de un usuario) que se quiere conectar con un puerto de alguien (contenedor)? TOMCAT
1º miapp.prod.es    > navegador ??? > DNS Externo > IP Balanceador EXTERNO
2º Mi nagevador ataca al IP Balanceador externo > IP PUBLICA DE UNA MAQUINA DE CLUSTER. Puerto? NodePort (>30000)
3º Al recibir eso una maquina N del cluster (>30000)  > netFilter > nombre de servicio (DNS Interno) Puerto del servicio
4º Nombre de servicio (Resuelto coreDNS)> IP de balanceo (IP Servicio)
    -- Quien hay detrás de esa IP? NADIE ... reglas en netFilter
5º El proceso hace una peticion a la IP de BALANCEO < Interceptada por el netFilter de QUE MAQUINA? De la de origen
6º Que hace netfilter? La pasa a un POD de destino



Cluster de Servidores WEB?                                                      > Deployment
    nginx \
    nginx  > Paginas html, js, css que ofrece al publico
    nginx /

Cluster de MariaDB| Kafka | ElasticSearch                                       > StatefulSet
    mariadb1    Ficheros de datos1      FILA-A      FILA-D   FILA-C
    mariadb3    Ficheros de datos3      FILA-B   FILA-A
    mariadb2    Ficheros de datos2              FILA-C   FILA-D  FILA-B



# Para que sirve un POD? En última instancia

Para tener agrupados contenedores

# Para que sirve un CONTENEDOR?

Para tener un sitio donde corra un proceso:
    - Para que realice un servicio
    - Que corra como demonio

TIPOS DE SOFTWARE QUE PUEDO EJECUTAR EN UN CLUSTER DE KUBERNETES
Demonio:    Un proceso que corre en segundo plano sin atender a nadie, el sabe que ha venido a hacer al mundo
            Abre algún puerto? NO .. porque no ofrece ningún servicio.
Servicio:   Un proceso que corre en segundo plano atendiendo peticiones de otros programas 
Script:     Un proceso que ejecuta una serie de tareas y ACABA !!!!!!



Cómo montamos un Demonio en Kubernetes? En un contenedor.. de un Pod... que configuramos igual que los pods que contienen servicios: Deployment, Statefulset, DaemonSet

Puedo correr en un contenedor un script? SI, sin problema... Y en un POD? NI DE COÑA !!!!!
    Por qué? Si kubernetes detecta que el pod no está corriendo... que piensa?
        ALARMA !!! DEFCON 7...8 MAXIMA ATENCION ROJO POR TODO SITIOS PROBLEMA!!!!!
    Un POD debe estar siempre corriendo: DEMONIOS y SERVICIOS
Existe un tipo de objeto en KUBERNETES/OPENSHIFT para ejecutar contenedores que contengan scripts: JOB

Ejemplo de script que puedo querer correr en el cluster de kubernetes?
- Backup de una base de datos


En KUBERNETES/OS :
Siempre que cree un objeto de tipo Deployment, Statefulset o Daemonset, voy a montar un objeto Service asociado?
NO... solo cuando los contendores del pod ejecuten servicios
Para que creamos un Deployment, Statefulset o Daemonset -> POD -> Contenedores <- Demonios y Servicios


Cómo cargo un Deployment, Statefulset, DaemonSet, Job, CronJob en Kubernetes / os?
    $ kubectl apply -f ficheroYAML - otros parametros
    $ helm install chart -- otros parametros
    $ oc apply -f ficheroYAML - otros parametros


## Determinar / Influir en el Scheduller para controlar donde (en que maquina del cluster) se instala(crea) un POD
¿Que ocurre ahi?
    Se crean los PODs... y aquí dentro? que implica esto?
        ¿Esto crea alguna regla en netfilter? NO... esto es cuando creamos un SERVICE
    Quién es el primero que se pone a trabajar? sheduller -> mira en que nodo, maquina? COMO LO DECIDE?
        Que mira el scheduller? 
            1 - Las que vienen en la plantilla del POD
                    - Afinidades y antiafinidades a nivel de POD
                    - Afinidades a nivel de Nodo
                    - Requisitos y limites de uso de recursos del pod
            2 - Los taints y tolerations de los nodos
            3 - El estado de los nodos
            4 - Otras reglas adicionales, directamente existentes en Kubernetes para influenciar al Scheduller:
                OBJETOS: LimitRange, ResourceQuota

## A nivel del Nodo
- Estado funcionamiento: ON , OFF
- Carga de trabajo / ocupación actual. Qué significa esto? Qué significa que esté saturado?
  - CPU al X% ? nop!!
  - RAM en uso al 70% nop!!!
  - El scheduller echa cuentas de qué? De lo que ese nodo ha comprometido.
- Recursos disponibles:
  - Nodo 1                                  POD A                           POD B                           LIBRE?
    - RAM: 32 Gbs                            garantizame: 10 Gbs RAM            garantizame 6 Gbs RAM          *16 Gbs de RAM*
    - CPU: 8 CPUs                                          4 nucleos            3 nucleos                       *1 Nucleo*
            REAL:        escenario 1                       7 Gbs                5 Gbs                           20 Gbs
                         escenario 2                      15 Gbs               12 Gbs                            5 Gbs
                         escenario 3                       4 nucleos            4 nucleos                        0 nucleos
    Lo que está libre... significa que si miro el estado actual del servidor, me encuentre 16Gbs libres? NO
                         que podría encontrar? Libres: 20 Gbs  Sin problema!  escenario 1
                                               Libres:  5 Gbs  Sin problema!  escenario 2
    Garantizame es: Si en un momento, el que yo quiera (que soy el POD) quiero mis megas, me los das. 
                    Y si no.. pues ahi los tienes...pa quien los necesite...

    Me llega un POD X que dice: Garantizame 16 Gbs de RAM. El scheduller puede pedir que se instale en el nodo 1?
        escenario 1: SI..... sino porque hay 16 Gbs NO COMPROMETIDO con nadie
                                                    ---------------
        escenario 2: SI... si no hay otra opción, sin problema: IMPACTO: 
            Falla al montar allí el POD X, si nada mas arrancar coge los 16Gbs? No falla....
                El pod X que culpa tiene... de que el POD A se haya flipado ! Decisión: Reinicio el POD A. 
                                                                                        No se pone ni colorao !
                MUCHO CUIDADO CON ESTO !!!!!!
                    Decisión: Siempre pongo como limite, lo que he pedido garantizado < REGLA DE ORO

    Que pasa si hago lo mismo con la CPU? No es lo mismo.
        Que pasa si una app / POD se pasa de uso de CPU del que ha solicitado que se le garantice?
            A priori se le deja (igual que con la RAM), si hay libre.
        Que pasa si entra un POD y se pone a querer usar las CPUs que a él se le han garantizado?
            Decisión: Le cierro el grifo al pod que se está pasando... empiezo a encolarle peticiones a la CPU...
                        Resultado, la app irá más lenta.
                POCO CUIDADO CON ESTO ... depende...

    En Kubernetes, para que arranque kubelet he de desactivar la swap a nivel de SO.

## A nivel del POD. Quien define un POD? Desarrollador

Pido resursos:
  - Quiero que me garantices  7 Gbs de RAM... normal? PUES CLARO !!!!!
  - Quiero que me limites a  10 Gbs de RAM... normal? NI DE COÑA !!!!!  POR SUPUESTO PONGO ESTO

Afinidades y antiafinidades a nivel de POD

Afinidades a nivel de Nodo
    Quiero esto en el nodo que tenga:
        - un role
        - labels

Los nodos llevan etiquetas. Algunas de serie... otras las ponemos nosotros como administradores:
    - lab, prod, qa, dev
    - tipoDisco: nvme, ssd, hdd ? Sentido? Y tanto.. no todos los datos son persistentes...
    - gpu: TITAN
    - zona: Europa, Africa, America
    - cliente: ClienteA
    - arquitectura arm x86
    -
    - A la hora de filtrar por etiquetas, podemos jugar con distintos operadors:
        - Que la etiqueta tenga un valor X
        - Que el nodo tenga la etiqueta, da igual el valor, pero que tenga la etiqueta
        - Que el valor de tal etiqueta se mayor (< >= <= ) que un valor
        - Tambien hay IN, NOT IN
      Además, cuando definimos esto, podemos pedir que sea obligatorio o preferido (PESO)
            - Quiero gpu SI, la que sea pero si y por narices
            - Prefiero disco: nvme: PESO 2
            - Prefiero disco: ssd : PESO 1

Quien define esto???  Quien pide esto? El desarrollador.

Mi trabajo como administrador:
    Tener los nodos con las etiquetas adecuadas
    Ofrecer ese conjunto de etiquetas a desarrollo: PUBLICAS

### SOLO PUEDO HACER ESO COMO Administrador? 

Esto sería poco: TAINTS / TOLERATIONS
Yo a mis nodos, les aplico unos TAINTS (Otro tipo de etiquetas)
    Cuando las pongo a un nodo, les digo en que momento se deben aplicar: OnSchedule, OnExecution, Siempre
Estas me las guardo bajo llave... No se las cuento a nadie: PRIVADAS

Si un desarrollador en un POD no ha puesto que tolera (TOLERATION) que el nodo tenga un TAINT, el POD no irá a ese nodo.
A quien me interese le diré... pon tal toleración.

FUNCIONALIDAD ADICIONAL: Esos taints se aplican también opcionalmente durante la ejecución de los PODS.
                        Si un taint ON ECECUTION Se añade a un nodo, todos los PODS que no tengan la TOLERATION al taint se
                        evacuan del nodo. INMEDIATAMENTE por kubernetes/OS.
                        Para que me puede servir? Mantenimiento de HW, SW (OS, Kb, Docker, SO)

## Otros ADMINISTRADOR <<<<<

Permiten establecer límites y valores por defecto de minimos garantizados sobre lo que los desarrolladores piden:

- LimitRange        Impone limites a nivel de POD
        - El máximo de uso de CPU de cada PODs que alli quieran poner, no puede superar 2 Gbs
        - El máximo uso de RAM de cada pods no puede superar 5Gbs
- ResourceQuota     Impone límites a nivel de Namespace
    En un namespace X: appFacturas prod: 
        - El máximo de uso de CPU entre todos los PODs que alli quieran poner, no puede superar 10
        - El máximo uso de RAM entre todos los pods no puede superar 32Gbs
        - Entre todos los volumenes que me pidan, no pueden pedir más de 100Gbs
        - No pueden pedir más de 3 volumenes aquí.

Pregunta? De esos 2 cuáles más importante para mi?
- Establecer un LimitRange    <<<<<< Para que sirve este? Arquitectura de mi cluster. Que no hagan un estropicio
            ESTE ES MAS RARO.. TENGO QUE ESTAR FINO y echar cuentas despacio... a ver que me interesa
    -  Tengo 50 máquinas de 8 CPUs: me interesa que me cojan pods de 5 CPUs
    -  Tengo maquinas con 32 Gbs de RAM: Limitado a 16 Gbs de RAM... necesitas más... Pon mas replicas de tus pods.
- Establecer un ResourceQuota <<<<<< Sin duda este 
    - No puedes en total de tus 5 pods que a lo mejor has creado pedir más de **20 cpus**







# Notas sobre la memoria maxima y garantizada. Equivalencia con Java

JAVA -Xmx1000m -Xms1000m                            Recomendación en JAVA, igual que en Kubernetes: IGUALES 
    limite     garantizado (con lo que arranco)

Tenga control de lo que hay ahi y tenga claro como se comporta
Maquina 32 Gbs
    App 1- 9-23 20 gbs   12
    App 2  23-9 20 gbs   12



# Producción Openshift / Kubernetes

N Maquinas... quiero hacer el mejor uso posible de ellas.
M aplicaciones en cluster que va desde 1 replica hasta 50 replicas.

Tengo HA si tengo 1 replica de una app? Si: Cluster Activo/Pasivo
Cuando antes configurabamos un cluster activo/pasivo, necesitaba 2 entornos duplicados. (desperdicio de recursos)
Escalado se hace dinámicamente de forma casi instantanea.
Me interesa tener pods pequeños. Muchos.

                %CPU        %MEMORIA   Incrementar escalado
App1:A      
App1:B
App1:C    
App1:D    
------------------------------------------------------------
                60%            70%

Autoescalado quien lo configura? Desarrollo / ADMINISTRADOR
10 PODS : 3 pods 70% --> Escalo arriba

Pod: 16 Gbs de RAM y 2 CPUs x 4
    RAM de una app?
        Datos de trabajo
        Caches se multiplican

App1:A      su juego de datos de trabajo... de las peticiones que ella está atendiendo  + CACHE \  % de duplicación enorme 
App1:B      su juego de datos de trabajo... de las peticiones que ella está atendiendo  + CACHE /

1 copia de la app           16 Gbs de RAM

2 copias                    12 Gbs de RAM
                            12 Gbs de RAM

REDIS, Oracle COHERENCE / Weblogic



App... necesita datos de la BBDD: App que vende entradas de conciertos. 
                                  App quiere en la RAM el listado de todos los conciertos que hay en un momento dado.
                                  Esos datos donde están? Donde tienen persistencia? Una BBDD

Aplicacion la tengo replicada en maquinas distintas: En las dos maquinas voy a tener en RAM el listado de todos los concierto

Si se cae alguien... en 1 minuto hay otro arriba.


Tradicional:                    CPU     MEMORIA           PETICIONES/min
App1-copia 1 < Servidor          25%      25%             1000 
App1-copia 2                     25%      25%             1000             
App1-copia 3                     25%      25%             1000             
App1-copia 4                     25%      25%             1000 

Tradicional:                    CPU     MEMORIA           PETICIONES/min
App1-copia 1 < Servidor          25%      25%             1000 
App1-copia 2                     25%      25%             1000             
App1-copia 3                     25%      25%             1000             
App1-copia 4                     25%      25%             1000 


Tradicional:                    CPU     MEMORIA           PETICIONES/min
App1-copia 1 < Servidor          25%      25%             1000 
App1-copia 2                     25%      25%             1000             
App1-copia 3                     25%      25%             1000             
App1-copia 4                     25%      25%             1000 


Si se caen 3... necesito que la que queda siga en marcha... porque el problemaes CUANTO ME VA A LLEVAR PONER LAS OTRAS 3 en funcionamiento

Probabilidad de que una falle? 1/100
2 maquinas: 2 fallen a la vez? 1/10000
3 maquinas: 3 fallen a la vez? 1/1000000
4 maquinas: 4 fallen a la vez? 1/100000000
Cluster 50 maquinas. 4/5 maquinas de reserva


Desarrollo me da el dato de RAM necesario? Me va pedir el garantizado? COMPLEJO 
De entrada, desarrollo no tiene ni idea de cuanta RAM necesita su sistema. 
El que desarrolla no tiene ni idea del uso real del sistema

MONITORIZACION ! Esta es la UNICA fuente fiable

Desarrollo en el pod me dice : Garantiza 3 Gbs de RAM... en el YAML que metemos en kubernetes/os < NO VALE PARA "NADA"
                                                                                            Es una medida de protección
Donde más debe estar esa info?Y ALLI EN PRIMER LUGAR:  En la configuración de la app.

arranca un app java: ES, Weblogic, Websphere, MySQL... yo le digo a cuanta memoria quiero que acceda.
                                                ^3 Gbs
                                                POD le garantizo 16Gbs... Estoy haciendo el tonto
¿Cuando para de coger RAM una BBDD?  NUNCA. Lo primero va a subir las tablas y los indices a RAM, Query RAM, y se mantiene en caches... Eh 1 tienes 4 Gbs
¿Cuando para de coger RAM una app JAVA? NUNCA. Garbage collection ...Cuando quiere...  Cuando ve que aquello va apretado

HELM < en ese YAML que se crea se dan 2 datos: 
    - Limite de memoria de la app en Kubernetes
    - Limite de memoria configurable en la app <> Garantizar lo que se ponga a la app/proceso (no al pod)



