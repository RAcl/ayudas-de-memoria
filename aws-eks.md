# Ayuda de memoria o breve howto

1. [Terminología de forma general](#terminología-de-forma-general)
2. [Preparando el Sistema](#preparando-el-sistema)
3. [Ejemplos de comandos para K8s](#ejemplos-de-comandos-para-k8s)
   1. [Configuración del equipo](#comandos-con-aws-para-configuración)
   2. [Uso de kubectx](#uso-de-kubectx-alias-y-cambio-de-cluster-a-administrar)
   3. [Uso de kubectl](#uso-de-kubectl)
      1. [Trabajar con manifiestos](#trabajar-con-manifiestos)
      2. [Revisar la infra](#revisar-la-infraestructura)
      3. [Namespace](#namespace)
      4. [Pod](#pod)
      5. [Deployment](#deployment)
      6. [Service](#service)
      7. [Set image](#set)
      8. [Rollout](#rollout)
4. [Autorizando a un usuario en el cluster](#autorizando-a-un-usuario-en-el-cluster)

## Terminología de forma general

- **nodo**: Maquina física que compone el cluster
- **cluster**: Conjunto de máquinas "físicas"
- **namespace**: Cluster virtual, el predeterminado se llama "default"
- **configmap**: espacio de archivos de configuración
- **pod**: Objeto que ejecuta un contenedor dentro de un namespace.
- **deployment**: Permite desplegar pods y gestionarlos
- **service**: Expone el servicio de un conjunto de pods
- **hpa**: Define el autoescalado horizontal de pods
- **PersistentVolume**: Define el volumen de almacenamiento persistente
- **ingress**: Define el balanceador
- **cronJob**: Tarea programada

## Preparando el sistema

Se debe instalar los siguientes comandos:

1. Instalar y configurar aws

   - Para instalar revisar https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

   ```bash
    aws configure --profile my-proj
   ```

   - más detalles en: https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html
2. Instalar kubectl

   - Revisar https://kubernetes.io/docs/tasks/tools/#kubectl
3. Instalar kubectx

   - Revisar https://github.com/ahmetb/kubectx#installation

## Ejemplos de comandos para K8s

### Comandos con AWS para configuración

1. Revisar que cluster están disponibles

   ```bash
   # revisar virginia
   aws eks list-clusters --profile my-proj --region us-east-1
   # revisar oregon
   aws eks list-clusters --profile my-proj --region us-west-2
   ```

   Regresará algo similar a

   ```json
   {
       "clusters": [
           "test-1-21",
           "eks-stack-dev"
       ]
   }
   ```
2. Incorporar la configuracion del cluster

   ```bash
   # Agregar a kubeconfig la configuración de virginia (staging)
   aws eks update-kubeconfig --profile my-proj --region us-east-1 --name eks-stack-dev
   # Agregar a kubeconfig la configuración de oregon (produccion)
   aws eks update-kubeconfig --profile my-proj --region us-west-2 --name eks-stack-prod
   ```

### Uso de kubectx, alias y cambio de cluster a administrar

1. Revisar el acceso a los cluster configurados en nuestra máquina

   ```bash
   # revisar los cluster
   kubectx
   ```
2. Crear alias de los cluster

   ```bash
   # crear alias produccion para eks-stack-prod
   kubectx produccion=arn:aws:eks:us-west-2:111222333444:cluster/eks-stack-prod
   # crear alias staging para eks-stack-dev
   kubectx staging=arn:aws:eks:us-east-1:111222333444:cluster/eks-stack-dev
   # revisar los cluster
   kubectx
   ```
3. Cambiar entre cluster

   ```bash
   # revisar los cluster
   kubectx
   # cambiarse al cluster de produccion
   kubectx produccion
   # revisar los alias del cluster
   kubectx
   # cambiarse al cluster de staging
   kubectx staging
   # revisar los alias del cluster
   kubectx
   ```

### Uso de kubectl

#### Trabajar con manifiestos

1. Aplicar recursos con un manifiesto
   ```bash
   kubectl apply -f manifiesto.yaml
   ```
2. Eliminar recursos con un manifiesto
   ```bash
   kubectl delete -f manifiesto.yaml
   ```
3. Revisando recursos creados con un manifiesto
   ```bash
   kubectl get -f manifiesto.yaml
   # o bien para mayor detalle
   kubectl describe -f manifiesto.yaml
   ```

#### Revisar la infraestructura

1. Listar cuantos nodos tengo

   ```bash
   kubectl get nodes
   # Retornará algo como:
   # NAME                           STATUS   ROLES    AGE    VERSION
   # ip-10-255-11-4.ec2.internal    Ready    <none>   147d   v1.22.12-eks-ba74326
   # ip-10-255-13-81.ec2.internal   Ready    <none>   147d   v1.22.12-eks-ba74326
   ```
2. Revisar la carga de los nodos

   ```bash
   kubectl top nodes
   # Retornará algo como:
   # NAME                           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
   # ip-10-255-11-4.ec2.internal    102m         5%     1371Mi          41%   
   # ip-10-255-13-81.ec2.internal   101m         5%     2414Mi          72% 
   ```
3. Describe el detalle de un nodo

   Para revisar el detalle de un nodo de debe aplicar ```kubectl describe nodes [nodo]```, en el detalle se podrán apreciar cosas como:

   - Arquitectura
   - Tipo de instancia
   - Region y Zona del nodo
   - Pods en ejecución en el nodo
   - Recursos ocupados

   Ejemplo:

   ```bash
   kubectl describe nodes ip-10-255-11-4.ec2.internal
   ```

   En caso de no especificarse el nodo listará el detalle de todos los nodos

#### Namespace

1. Listar los namespace
   ```bash
   kubectl get namespace
   # o bien
   kubectl get ns
   ```

#### Pod

1. Listar los pod de un namespace

   Para listar los pod se ejecuta
   ```kubectl get pod -n [namespace]``` o bien ```kubectl -n [namespace] get pod```
   donde **namespace** es uno de la lista del comando anterior

   ```bash
   kubectl -n staging get pod
   ```

   ***Cosas importantes a revisar son***:

   - Columnas `READY` y `STATUS`, indican el estado del pod, 0/1 indica detenido, 1/1, activo
   - Columna `RESTARTS`, indica cuantas veces se ha reiniciado el pod intentando arrancar con éxito, lo normal es el valor 0
   - Columna `AGE` indica aproximadamente desde hace cuanto se está ejecutando el pod
2. Log de un pod

   Para revisar el log del pod **test-my-proj-846c86fb46-b52h7** del namespace **staging** ejecutamos:

   ```bash
   kubectl -n staging logs test-my-proj-846c86fb46-b52h7
   # o bien con -f para dejar el log escuchando, termina con <Ctrl><C>
   kubectl -n staging logs -f test-my-proj-846c86fb46-b52h7 
   ```

   donde el nombre del pod **test-my-proj-846c86fb46-b52h7** lo obtenemos del comando anterior
3. Acceder a un pod

   Iamginemos que necesitamos entrar a un pod a revisar algo, por ejemplo, si desde el pod llego a la BD. En el ejemplo, el comando a usar, suponiendo que se trata del mismo pod que revisamos en el log, es:

   ```bash
   kubectl -n staging exec -it test-my-proj-846c86fb46-b52h7 -- bash
   ```

   No siempre está disponible la shell bash, en algunos casos puede invocarse la shell "sh" o bien puede ser necesario entregar el path completo, por ejemplo, "/usr/bin/bash".

#### Deployment

1. Listar los deployment de un namespace

   Para listar los deployment se ejecuta
   ```kubectl get deployment -n [namespace]``` o bien ```kubectl -n [namespace] get deployment```
   donde **namespace** es uno de la lista del comando ```kubectl get ns```

   ```bash
   kubectl -n staging get deployment
   ```
2. Describir un deployment

   Por ejemplo para describir el deployment de **test-my-proj** en el namespace **staging** ejecutamos:

   ```bash
   kubectl -n staging describe deployment test-my-proj
   ```

   ***Cosas importantes a revisar son***:

   - `Image`: entrega la imagen del deployment aplicado
   - `NewReplicaSet`: entrega el numero de réplicas que se deben crear y quien lo solicita
   - `Limits`: cuanta cpu y memoria puede llegar a usar
   - `Environment`: Variables de entorno
   - `Mounts`: path o punto de montaje si es que existe un recurso externo de disco
   - `Volumes`: recursos externos de disco
3. Log de un deployment

   Para revisar el log del deployment **test-my-proj** del namespace **staging** ejecutamos:

   ```bash
   kubectl -n staging logs deployments/test-my-proj
   # o bien con -f para dejar el log escuchando, termina con <Ctrl><C>
   kubectl -n staging logs -f deployments/test-my-proj
   ```

   Al revisar el log del deployment se revisa lo llegado a todos los pod del deployment
4. Actualizar la imagen de un deployment

   Imaginemos que tenemos un nginx en version 1.9.0 y queremos pasar a la 1.9.1 en el namespace **staging**, el comando sería

   ```bash
   kubectl set image deployment/nginx nginx=nginx:1.9.1 --record deployment/nginx -n staging
   ```

   Para revisar el estado del despliegue

   ```bash
   kubectl rollout status deployment/nginx -n staging
   ```

Para más detalles sobre el deployment se aconseja revisar la [documentación oficial][deployment-doc]

#### Service

1. Listar los services de un namespace

   Para listar los services ejecutamos ```kubectl get services -n [namespace]``` o bien ```kubectl -n [namespace] get services```

   Por ejemplo para listar los services en el namespace **staging** ejecutamos:

   ```bash
   kubectl -n staging get services
   ```
2. Describir un servicio

   Por ejemplo para describir el servicio de **test-my-proj** en el namespace **staging** ejecutamos:

   ```bash
   kubectl -n staging describe service test-my-proj
   ```

   ***Cosas importantes a revisar son***:

   - `TargetPort`: puerto del expuesto por el contenedor
   - `Port`: puerto por el que el servicio será expuesto
   - `Endpoints`: pod que está exponiendo

   Para revisar los pod de endpoint del comando anterior, podemos correr el comando ```kubectl -n staging get pod -o wild``` que detallará entre otras cosas la IP (-o wild agrega una visa "ancha" o con un poco más de detalles, existen recursos en que con esta opcion o sin ella enregan la misma cantidad de detalles).

> #### kubectl get/describe
>
> En general se puede cualquier recurso revisar con get y describe, además es posible agrupar recursos, por ejemplo, para ver los recursos _pod, deployment, service, replicaset, horizontalpodautoscaler, cronjob, job,ingress, persistentvolumeclaim_ y _configmap_ del namespace **staging**, con el comando:

```bash
kubectl get all,ingress,pvc,cm -n staging
```

#### Scale

Se puede escalar de forma incremental o decremental fijando una nueva cantidad de pods de un deploy con ```scale```, indicando la cantidad de replicas con las que debe quedar.

```bash
kubectl scale deployments/[miDeploy] --replicas=4 -n [namespace]
```

#### Set

Es posible actualizar la imagen de un deployment simplemente detallando cual es la nueva imagen a utilizar.

```bash
kubectl set image deployments/[mideploy] [template]=[ecr-image]:[version] --record=true -n [namespace]
```

#### Rollout

1. Seguimiento de una Actualización

   Se puede seguir una actualización con ```rollout status``` de la siguiente forma:

   ```bash
   kubectl rollout status deployment/[mideploy] -n [namespace]
   ```
2. Reiniciar un deployment

   Se puede reiniciar los pod de un deployment con ```rollout restart``` usandolo de la siguiente forma:

   ```bash
   kubectl rollout restart deployment [mideploy] -n [namespace]
   ```
3. Ver el historial de cambios

   Con ```rollout history``` se puede revisar el historial de cambios

   ```bash
   kubectl rollout history deployment/[mideploy] -n [namespace]
   ```

   Para ver los detalles de la revisión **N**, ejecute:

   ```bash
   kubectl rollout history deployment/[mideploy] --revision=[N] -n [namespace] 
   ```
4. Regresar a la versión anterior

   Despues de aplicar un ```set image``` se puede regresar a la versión anterior ejecutando un ```rollour undo``` de la siguiente forma:

   ```bash
   kubectl rollout undo deployment/[mideploy] -n [namespace]
   ```
5. Regresar a una versión amterior en específico

   De forma similar a lo anterior, tras ejecutar el ```rollout history```, uno puede optar por regresar una revisión numero **N**:

   ```bash
   kubectl rollout undo deployment/[mideploy] --to-revision=[N] -n [namespace]
   ```

## Autorizando a un usuario en el cluster

Para autorizar a un ususrio en el cluster se debe hacer lo siguiente:

1. Crear al usuario en la consola de aws
2. Asegurar en la consola de aws que el usuario tenga los permisos y roles necesarios
3. Editar el configmap aws-auth del namespace kube-system

   ```bash
   kubectl edit -n kube-system configmap/aws-auth
   ```

   Agregar en **data** la variable **mapUsers** el usuario con el atributo **userarn** con el arn del usuaurio de la consola aws, el atributo **username** que es el nombre de usuario en la consola de aws, y el atributo **groups** con **system:masters**.

   Debe quedar similar a:

   ```yaml
   mapUsers: |
     - userarn: 'arn:aws:iam::123456XXXXXX:user/john.doe'
       username: 'john.doe'
       groups:
         - 'system:masters'
   ```

   Para agregar más usuarios se agregan bajo **mapUsers** en estructura similar

   Para revisar la edición del archivo de configuración puede ejecutar

   ```bash
   kubectl get -n kube-system configmap/aws-auth -o yaml
   ```

---

~~ RAcl : 2023
