Instalar RedHat Openshift:
- Cloud AWS
- Cloud Azure
- Cloud IBM
- Cloud Redhat

Instalarlo sin gestión (Donde quiera)

---
NORMAL DE UN CLUSTER

POD1                POD2
nginx  --- NP -->   mariadb

Driver RED es que el que aplica las reglas:
    NetworkPolicy

----

Istio / Linkerd SERVICE MESH dentro del cluster

POD1                POD2
nginx               mariadb
  V                   ^
proxy (envoy) ----> proxy (envoy)
    NP         TSL      NP
    cert                cert 


POD 1: 3 contenedores:
    initContainer: alta reglas de red:
                    iptables a netfilter
    nginx
    proxy

---

Maquina 1
    kubelet -> docker
    kubeproxy -> meter reglas en netFilter
    PODs - IP POD 1
        initC-> dar de alta reglas en netfilter
                pero sin tener en cuenta al kubeproxy
        nginx
        proxy-envoy IP POD 1

Maquina 2
    kubelet -> docker
    kubeproxy -> meter reglas en netFilter
    PODs - IP POD 2
        mariadb
        proxy-envoy IP POD 2


---
Cluster con muchos servicios dentro

Cluster con 4 servicios... o 20 ... esto no me vale
    Trabajo con NetworkPolicies


Cluster con mas de 50 servicios - AUTO ISTIO
BBDD, ELASTICSEARCH

                                follon/lio
Cluster de kubernetes SERVICES MESS
            ISTIO              MESH
                                malla organizada




----
# Acceso a Servicios externos al cluster/ Exposición de servicios

Necesitamos acceder externamente a un servicio del cluster: 3 formas:
    Servicios NodePort
    Servicios LoadBalancer
    Ingress/Route -> ClusterIP


# Route / Ingress

Se usa un proxy reverso instalado dentro del cluster: IngressController
ROUTE O INGRES es una regla de configuración para ese proxy reverso ~ VirtualHost apache

URL PUBLICA
    - http(s):// servidor : PUERTO / RUTA
        http: 80
        https: 443
    - http(s):// servidor / RUTA

En mi cluster tendré 50 apps:
    Subdominios:
        https://app1.micluster.com
        https://app2.micluster.com
        https://app3.micluster.com
    Paths:
        https://micluster.com/app1
        https://micluster.com/app2
        https://micluster.com/app3    
    Hibrido
        https://app1.micluster.com              -> Produccion
        https://app1.micluster.com/desarrollo   -> Misma app pero en ns desarrollo
        https://app1.micluster.com/test         -> Misma app pero en ns integracion

Quien gestiona el DNS publico? YO

Datos que hacen falta:
    DOMINIO  app1.micluster.com
    PATH     /desarrollo
    SERVICIO DENTRO DEL CLUSTER (clusterIP) al que se redirigen las peticiones
        A QUE PUERTO DEL SERVICIO 8080

Route:
    TLS: Si lo quiero exponer por https
        Dependiendo de quien haga la encriptación y quien da la cara
            Servicio backend - Passthru
            El proxy reverso - Edge         (Dentro del cluster la comunicación es http) MAL ASUNTO
            El proxy reverso habla con el cliente por https con certificaco propio     - Re-encrypt       
                La comunicación entre el proxy resverso y el servicio interno es vía https también, con certificado propio del servicio

----
Lo primero desde fuera ver que llego al PROXY REVERSO
----

Miro el pod: READY, consigo tocar al puerto:
    http:   curl
            telnet mysql

Miro a nivel de servicio a ver si llego. Si no llego: 
    - Network - policy
    - Que no tenga bien el servicio configurado (puerto mal puesto o etiqueta)

Si llegas hasta ahi: INGRES CONTROLLER log
    Mirar la regla del ingress controler -> Servicio + Puerto + NOMBRE + PATH


------------ ------------ ------------ ------------ ------------ ------------ 
# Instalaciones de aplicaciones dentro del cluster
------------ ------------ ------------ ------------ ------------ ------------ 
Cluster de kubernetes / openshift es infraestructura
    - Maquinas
    - Redes
    - Politicas de red
    - Balanceos de carga
    - Dns
    - Seguridad
    - Limitación de recursos a hardware

Instalar Pods > Contenedores > Ejecutan procesos / apps

# Cómo instalamos apps?

- Con ficheros YAML que vamos cargando kubectl o oc... o traves del interfaz grafica . PRACTICO? 
    Las cosas las tengo controladas... y eso está bien.. pero práctico.. práctico no lo veo.. Es un poco rollo.

    Una app la voy a queres instalar varias veces previsiblemente? Actualziaciones... distintos entornos (pruebas, desarrollo). Que acabo con tropetantos ficheros yaml? desarrollo, produccion, integracion.. cliente 1, cliente 17...

- HELM: Plantilla personalizable que permite instalarse automaticamente en el cluster, con soporte para actualizaciones
    También necesito varios ficheros... uno por entorno, cliente... 
    Pero solo 1 fichero por cada uno, y muy sencillo, ya que solo contiene valores de configuración.
        helm.sh < charts = plantillas (versiones)
        Operativa: 
        1.  Instalamos helm en la maquina (Ejecutar un script)
        2.  Descargar chart de repos (que he de dar de alta)
        3.  Modifico el fichero values.yaml
        4.  Instalamos
            $ curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
            $ helm repo add gitlab https://charts.gitlab.io/
            $ helm pull gitlab --untar
            $ helm pull gitlab/gitlab --untar
            $ cd gitlab/
            $ cat values.yaml
            $ helm install NOMBRE gitlab/gitlab -f VALUES.yaml -n NAMESPACE --create-namespace
            $ helm upgrade NOMBRE gitlab/gitlab -f VALUES.yaml -n NAMESPACE --create-namespace

Mariadb



- Operadores: Una forma de instalar apps dentro del cluster

Un operador genera un conjunto de los que denominamos "crd"
crd => Custom Resource Definition
        Básicamente un nuevo tipo de objeto que cargar en Kubernetes/Openshift
        YAML: 
            kind: Jenkins
            apiVersion: jenkins.operator/v1
            metadata:
                name: Nombre de mi instalacion de jenkins
            spec: 
                # Depende del objeto que estoy cargando
                # Hablamos en el el lenguaje de la aplicación



# Seguridad

Cualquiera que necesite conectar con un cluster de Kubernetes o de Openshift necesita de un Service Account.
Cualquiera:
- Persona accediendo a través del cli oc/kubectl o de Dashboard web
- Programa que quiera hacer cosas en el cluster
  - En Kubernetes, para que un programa se ejecute dentro del cluster (POD) no necesita un service account
  - En Openshift, para que un programa se ejecute dentro del cluster (POD) SI necesita un service account
    - Por defecto los PODs no pueden usar el usuario root

    Ejemplo: CERT-MANAGER, PROVISIONER
        Programa que genera certificados con let's encrypt para los ingress...
        Y me estoy ejecutando dentro del cluster.
        Y lo que quiero es mirar cuando hay un nuevo ingress
        Y en ese momento generar un secret que contenga el certificado que acabo de solicitar a let's encrypt

Un Usuario en Openshift es un tipo de Un service Account

Un service Account es un objeto más que creamos dentro del cluster... YAML

A los service account (y a los usuarios) les añado roles. Esa operación es un Role-Binding.
    OJO: los roles los añado a nivel de NAMESPACE
A los service acount (y a los usuarios) tambien les puedo añadir roles a nivel gloval, no asociados a un Namespace.
    Por que dijimos que hay objetos que no se gestionan a nivel de namespace: PersistenVolume
    Ahi nos surgen los canceptos de Role Global  (ClusterRole) y Asignación de Role Global (ClusterRoleBinding)

Un Role es un conjunto de privilegios / RULES (reglas):
    - TIPO DE OBJETO DEL CLUSTER: pods, secrets, configmap
    - VERBOS: VERBS: Acciones puedo realizar: create, delete, get, update


Los usuarios de Openshift se gestionan en un proveedor de identidades externo:
    - LDAP (activeDirectory)
    - Github
    - Gitlab
    - KeyCloak <LDAP>