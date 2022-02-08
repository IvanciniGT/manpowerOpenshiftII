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
- Deployment
- StatefulSet
- DaemonSet
- PersistentVolume
- PersistentVolumeClaim
- Service                                           Que es? 
                                                      - ClusterIP:      Entrada en el DNS de Kubernetes
                                                                        Balanceo de carga entre PODs (reglas de netFilter)
                                                      - NodePort:       = ClusterIP
                                                                        + Otra regla en netFilter adicional (NAT) > 30000
                                                      - LoadBalancer    = NodePort
                                                                        + Configuración automática de un Balanceador Externo:
                                                                            Entre que balancea el balanceador externo?
                                                                                Balancea petiones externas al cluster
                                                                                entre los nodos del cluster
                                                    Sirve para publicar lo que ofrece un POD (o varios en cluster) a través de un puerto al resto del cluster o incluso al exterior del cluster
- Namespace / Project
- ConfigMap
- Secret
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