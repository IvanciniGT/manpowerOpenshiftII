

Cluster:
    Pods: Cada uno tiene su propia IP
        Contenedores. Que heredan la IP del Pod.
            Los contenedores de un POD no pueden abrir el mismo puerto... ya estaría en uso
    Nodos: Cada uno tiene su IP
    Services: Cada servicio también tiene su IP.
        Esa dirección IP no la tiene asignada ningún programa... Es procesada por?
            El netFilter de cada host del cluster


# TRABAJOS A REALIZAR:

Nuestro objetivo es tener un app dentro del cluster en funcionamiento y accesible desde internet.

## Pasos:

1º Creado Kub:Namespace | OS: project                           √
2º Limitarlo: <ResourceQuota> + LimitRange opcional             ·
Nº Cargar un Template > yaml
    Deployment | StatfulSet | DaemonSet
        PodTemplate
            Contenedores

# Terminal:
#  $ kubectl exec -it POD -c CONTENEDOR -n NAMESPACE -- COMANDO
#                                                       /bin/bash


# service account
Usuario que puede hacer cosas en el cluster / Contra el cluster

Kubernetes: ServiceAccount Cuenta con la que conectarnos a un cluster para hacer operaciones
OpenShift:  ServiceAccount & User
    ServiceAccount se limitan a programas. PROVIDER (el que crea volumenes persistentes en el cluster en auto)
        Token
    User: Es un service account que va a ser utilizado por un humano
        Contraseña

# Cómo miro que está pasando en el cluster en un proyecto

$ kubectl get all -n PROYECTO
$ kubectl describe service SERVICIO -n PROYECTO
    --> PODS backend... malo será que no haya ninguno
    --> telnet curl puerto del servicio a ver contesta
$ kubectl describe deployment DEPLOYMENT -n PROYECTO
    $ kubectl describe replicaset REPLICASET -n PROYECTO
$ kubectl describe statefulset DEPLOYMENT -n PROYECTO
$ kubectl scale deployment DEPLOYMENT -n PROYECTO --replicas=3

$ kubectl describe pod POD -n PROYECTO
$ kubectl logs POD -c CONTENEDOR -n PROYECTO    

$ kubectl get all -n PROYECTO -o wide # Información extendida (IPs)
# OJO: El get all... realmente no devuelve todo: service, ingress, routes, deployments, statefulset, daemonset, pods, cronjobs, jobs, replicaset, hpa
# No me devuelve ni los resource quota, secret, configmap, pvc

# $ kubectl label node NODO ETIQUETA=VALOR
# $ kubectl label node worker1 disco=rapido

Por otro lado... en mi deployment A...
    Tengo un affinity node: disco=rapido
    Tengo un nodeSelector: disco=rapido

¿con esto garantizo que mi máquina solo tenga pods creados desde ese deployment A... que es al único que le he puesto esa etiqueta en las afinidades (a ningún otro deployment le he puesto eso)? 
NO... por qué?

Que el deployment A (sus pods) siempre vayan a un nodo con esa etiqueta. √

Para garantizar eso.. que haríamos?
Si o quiero que un nodo solo tenga pods de un tipo.
Nodo exclusivo.

Taints/Toleraciones : (Nodos dedicados | Evacuar nodos)
Estas me imponían reglas a los nodos:
    Taint= Permitete rechar ciertos pods

# REDES Y COMUNICACIONES

namespace (project) bbdd-produccion-app1
namespace (project) bbdd-produccion-app2

namespace : app2 -> bbdd-produccion-app2. Podría acceder a la bbdd-produccion-app1? A priori SI. Y ESTO ES UN PROBLEMA. NO LO QUIERO

NETWORK POLICY


