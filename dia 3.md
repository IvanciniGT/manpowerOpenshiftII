Nuevo cliente Mi cluster:
1º Crear namespace
2º Meter la tijera. Limitarle a nivel de NS
3º Meter la tijera. Limitarle a nivel de POD / Contenedor
4º Permisos....
5º Donde va a ejcutar sus pods... En ocasiones lo rellenará él... en otras yo.
    Normalmente el trabajo del equipo de desarrolloacaba creando un:
        CHART DE HELM = Plantilla de despliegue = ENTREGABLE
    Ahí dentro vamos a meter un montón de cosillas:
        - Donde va aquello en la infra
        - Monitorización
    De quién es responsabilidad esto? Del desarrollador? MIA: ADMINISTRACION






# Cuando un desarollador quiere usar un VOLUMEN PERSISTENTE

Que hace el desarrollador? Crea un pvc (PersistentVolumeClaim): PVC-17

Que definimos en un PersistentVolumeClaim:
    Tipo de almacenamiento: storageClass    A
    Caracteristicas:        capacity        10Gi

Esto existe para DESACOPLAR las labores de un DESARROLLADOR y un ADMISNITRADOR
    por otro lado, para que un volumen realemnet SEA PERSISTENTE

El desarrollador dice que un POD: POD-34 usa el volumen asignado al PVC-17.

Que necesitamos ahora en Kubernetes/OS? Un pv (PersistentVolume): PV-99

Qué definimos cuando creamos un PersistentVolume:
    Tipo de almacenamiento: storageClass    A
    Caracteristicas:        capacity        20Gi
    Dónde está realmente ese volumen: 
        AWS-S3-<Id>   
        AZURE    
        NFS(192.168.2.189://datos/volumen15)

A partir de aquí, qué ocurre en kubernetes? 
    Quien junta PVC con el PV: Kubernetes      PVC-17 <> PV-99
        En base a qué? A que el PV satifaga las demandas del PVC

POD-34 <> PVC-17 <> PV-99 *** Por qué necesito el conpto de pvc para la persistencia?
Si desaparece el pod-34... el pvc sigue existiendo y sigue vicnulado al PV
          PVC-17 <> PV-99
El dia de mañana (o en 10 segundos), se creará un pod nuevo: POD-18, que puedo decirle que use el volumen del PVC-17
POD-18 <> PVC-17 <> PV-99
El pod que quiera usar un pvc ... debe estar en el mismo Namespace que el pvc

Quien crea los PV en kubernetes. 2 opciones:
    - Administrador. A manita... creando un fichero YAML ** Problema? 
                                - me puedo confindir... ya puedo ir con cuidado
                                - lo hago antes de que lo pidan. Construyo un poll de volumenes
                                    - Estoy tirando euros
                                - lo puedo hacer despues de que lo pidan... demora
    - Provisionador de volumenes, creado y configurado por el admisnitrador
      Esto es un programa que corre dentro de kubernetes y que está atento de todos los pvc nuevos que se crean por los desarrolladores.
      En automatico crea un pv para satsfacer la petición. ESTO ES LO GUAY


VOLUMENES:                                                              TIPOS
- NO PERSISTENTES
    Se usan mucho? Más que los otros. Para qué?
    - Inyectar ficheros a contenedores: Configuración / secrets         configmap / secret
    - Compartir datos entre pods                                        emptyDir




---
kind:           PersistentVolumeClaim
apiVersion:     v1          

metadata:
    name:       peticion-volumen-base-de-datos
    
spec:
    resources:
        requests:
            storage: 10Gi # Tamaño que necesito
    # Tipo
    storageClassName: rapidito-redundante
    accessModes:
        - ReadWriteOnce           # Ese volumen / pvc solo se puede usar por 1 Máquina /En paralelo
#       - ReadWriteOncePod        # Ese volumen / pvc solo se puede usar por 1 POD /En paralelo
#        - ReadWriteMany          # Ese volumen / pvc se puede usar por N Máquina /En paralelo
#        - ReadOnceMany           # Ese volumen / pvc se puede usar por N Máquinas para lectura /En paralelo

---
kind:           PersistentVolume
apiVersion:     v1          

metadata:
    name:       volumen-fedora
    
spec:
    # Morralla... Información descriptiva
    capacity: 
        storage: 20Gi 
    storageClassName: rapidito-redundante
    accessModes:
        - ReadWriteOnce
        - ReadWriteMany
        - ReadOnlyMany
    persistentVolumeReclaimPolicy: Delete   # Retain
    
    # Donde esta esto físicamente
    # Tipo de volumen
    hostPath:    # PARA JUGAR
        path: /home/ubuntu/environment/datos/persistente
        type: DirectoryOrCreate 






BBDD | KAFKA | ELASTICSEARCH

Cluster:

Nodo 1
    dato1           dato3
Nodo 2
    dato1   dato2   dato3
Nodo 3
            dato2

Replicación:

Maestro - Activo < INSERT 
    Datos de este 
Espejo
    son iguales en este

    Pero eso te lo aegura el motor de BBDD




POD: mysql-1
v5.7.1
Volumen: el de la pvc: volumen-mysql

POD: apache-1
v2.4.23
Volumen: el de la pvc: volumen-apache

POD: mysql-2
v5.7.2
Volumen: el de la pvc: volumen-mysql

PVC: volumen-mysql
10 Gbs
rapidito y redundante

PV: volumen-1587-aws-s3
25 Gbs
rapidito y redundante

volumen-1587-aws-s3 <> volumen-mysql <> mysql-2