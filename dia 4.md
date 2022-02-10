# TAINTS y TOLERATIONS

Influir al scheduller en qué pods instalar o no en un nodo. 
Más orientadas a administradores.

Pods (afinidades contro otros PODs o con NODOS y antiafinidades solo a nivel de POD)
    Que un POD se acerque o aleje de un NODO.

$ kubectl label node NODO LABEL=VALOR
$ kubectl label node NODO LABEL=VALOR-          # Quitar el label

Nodo (Taints y Tolerations)
    Que un NODO decida que PODS quiere dejar que se acerquen o que sean rechazados

$ kubectl taint node NODO TAINT=VALOR:efecto
$ kubectl taint node NODO TAINT=VALOR:efecto-   # Quitar el taint

## efecto:
- NoSchedule                Evitar que un pod sea planificado en un nodo
- PreferNoSchedule          Tratar de evitar que un pod sea planificado en un nodo
- NoExecute                 Aunque el pod haya sido planificado en es nodo, será evacuado
                            Tiene sentido solamente para cambios en caliente 

## Los TAINTs tienen 2 usos:
- Evacuar máquinas: NOEXECUTE
- Tener maquinas dedicadas << Esto no se hace con labels
  - Maquina con GPU 

### Maquinas dedicadas

Pod pedirá maquina con GPU LABEL -> Genero afinidad y me pongo en laquina con GPU
Pos no pide máquina con GPU      -> Pues quizás tambien llego a esa máquina

Nodo 1
    label: tipo-disco: rapidito
Nodo 2
    label: tipo-disco: rapidito
    label: gpu:        true

Pod: afinidad con nodo (label: gpu: true)                    -> Nodo 2
Pod: afinidad con nodo (sin reglas de afinidad)              -> Nodo 1, Nodo 2
Pod: afinidad con nodo (antiafinidad si tiene el label gpu)  -> Nodo 1, Nodo 2
    Esto directamente no se puede hacer... aun así: El desarrollador o usuario del cluster
    no piensa en eso

Nodo 1: RECHAZA TODO AQUELLO QUE NO TENGA UNA TOLERANCIA A TU TAINT.
Nodo: Taint: tieneGPU:TRUE:NoSchedule

Pod: TOLERACION: tieneGPU:TRUE:NoSchedule

# Entre los taints, hay algunos gestionados por Kubernetes/OS
node.kubernetes.io/not-ready        Significa que el nodo aún no está listo
node.kubernetes.io/unreachable      No se nada de él
node.kubernetes.io/memory-pressure
node.kubernetes.io/disk-pressure     


# De repente pierdo conexión de red con una máquina.

Kubernetes en auto: TAINT node.kubernetes.io/unreachable:NoSchedule
    Ningún pod nuevo irá allí.
Y los que estén allí corriendo. A priori... allí siguen. Kubernetes no hace nada con ellos.
Puede suponer un problema. 
Solución rápida: TAINT node.kubernetes.io/unreachable:NoExecute:
    Sacar lo que hay dentro....

¿Qué es el nodo? Dentro de la bbdd (etcd) kubernetes una entidad... que hace referencia a ese nodo.
Para kubernetes los pods de allí los saca... ¿Que es sacar? 
    ¿qué es mover? = BORRADO del antiguo Y CREACION de uno nuevo > Scheduler


# Cómo sabe o decide Openshift que un POD está en funcionamiento o no?

Mirando el PID que sigue vivo. Si se cae... y el contenedor está en un POD, se reinicia el POD.
                                              si está lanzado desde un JOB (scripts) se anota

Nos vale esto en un entorno de PRODUCCION? no. Esto es un básico ... esto por supuesto
Pero... contesta a la gente si es servicio, por ejemplo.
En kubernetes / Openshift configuramos Probes de vida.
    StartupProbe     curl localhost:8080   $?!=0        Si falla? Se reinicia por Kubernetes/OS
    LifenessProbe                                       Si falla? Se reinicia por Kubernetes/OS
    ReadynessProbe                                      Si falla? Se saca del backend del 
                                                                  servicio asociado
        Life ... estará haciendo algo.
        Ready... está listo para dar servicio
        Mysql.. que entre en modo admin.> backup... y no atiende peticiones
            Life ... no quiero que se reinicie
            NO Ready... no lo meto en el servicio para hacer balanceo sobre él.

# ¿Cómo sabe docker o herramienta equivalente si un contenedor está o no en funcionamiento?

Mirando el PID que sigue vivo. Si se cae.. lo anota



# En entorno reales.

No tenemos 1 cluster por producto. Tenemos 1 cluster para un huevo de proyectos.

2 cluster 50 maquinas replicados ... 200 maquinas ... Un huevo de apps.

Un equipo de superadministracion: root

Un equipo administrador por conjunto de apps


Desarrollo -> Entregable -> Equipo devops -> Dockerizar -> Kubernetes -> Admin


Kubernetes | "Openshift"



50 microservicios hablando entre si escalados.. 200 pods + 200 proxy (envoy)

httpS: crear clave publica/privada en cada maquina 
                    publica -> Exportada en un certificado
                    entidad certificadora
                    Todo esto hay que renovarlo cada X tiempo 
                    Configura en cada servidor de apps que admita y conzca al resto de nodos 

ISTIO < 5 minutos      > Man In the middle

# Evitar Brain spliting
Nodo1           Nodo2