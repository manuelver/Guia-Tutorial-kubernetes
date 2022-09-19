# Índice
- [Componentes](#componentes)
- [Recursos Kubernetes](#recursos-kubernetes)
- [Instalación kubctl y primeros pasos](#instalación-kubctl-y-primeros-pasos)
- [Manifiestos de Pelado Nerd](#manifiestos-de-pelado-nerd)
- [Cheatsheet kubernetes](#cheatsheet-kubernetes)
- [Ejemplo de un YAML para un pod básico de busybox](#ejemplo-de-un-yaml-para-un-pod-básico-de-busybox)
- [Agradecimientos](#agradecimientos)


# Guía Kubernetes

Kubernetes es un sistema de código libre para la automatización del despliegue, ajuste de escala y manejo de aplicaciones en contenedores que fue originalmente diseñado por Google y donado a la Cloud Native Computing Foundation (parte de la Linux Foundation). Soporta diferentes entornos para la ejecución de contenedores, incluido Docker y su misión es la orquestación de dichos contenedores.

Es declarativo.
![](img/kubernetes-declarativo.png)

## Componentes
![](img/Componentes_kubernetes.png)

- **etcd** - Guarda el estado de Kubernetes
- **servidor API** - Es un componente central y sirve a la API de Kubernetes utilizando JSON sobre HTTP, que proveen la interfaz interna y externa de Kubernetes.
- **Planificador** - Es el componente enchufable que selecciona sobre qué nodo deberá correr un pod sin planificar basado en la disponibilidad de recursos.
- **Administrador de controlador** - Es el proceso sobre el cual el núcleo de los controladores Kubernetes como DaemonSet y Replication se ejecuta.
- **Nodo** *(esclavo o worker)* - Es la máquina física (o virtual) donde los contenedores (flujos de trabajos) son desplegados. 
- **Kubelet** - Es responsable por el estado de ejecución de cada nodo, es decir, asegurar que todos los contenedores en el nodo se encuentran saludables.
- **kube proxy** - Reenvía el tráfico al pod donde tiene que ir.
- **cloud controler manager** - Es el encargado de conectarse a la API del cloud. <u>Es la diferencia con otros orquestadores</u>. Permite correr los contenedores y hacer funciones extras que no estén instaladas en nuestro espacio.
- **cAdvisor** - Es un agente que monitorea y recoge métricas de utilización de recursos y rendimiento como CPU, memoria, uso de archivos y red de los contenedores en cada nodo. 

No es buena idea correr tráfico de clusterización en equipos personales.


---
## Recursos Kubernetes

![](img/Recursos-kubernetes.png)

Los **Pods** son los contenedores de Kubernetes (puede tener varios contenedores). La unidad mínima de computación. Comparte una única IP. Generalmente por cada pod correrá un contenedor, pero puede darse las circunstancias de que sea preferente correr varios contenedores en un pod.

**ReplicaSet** son quienes se aseguran que los contenedores siempre se están ejecutando. Asegura
- Que no haya caída del servicio
- Tolerancia a errores
- Escalabilidad dinámica

**Deployment** es responsable de las actualizaciones de los contenedores y despliegues automáticos de la aplicación.. Maneja los ReplicaSets.

Los **servicios** permiten los accesos a la aplicación.

**Ingress** es un proxy inverso que se comunica con los servicios y nos da la posibilidad de tener un nombre DNS, balanceo de carga entre pods...

Los **namespace** son clusters virtuales respaldados por el mismo clúster físico.


---
## Instalación kubctl y primeros pasos

Lo primero es instalar kubectl.

*Documentación oficial: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/*

Descargamos los paquetes.
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```
Descargamos validador kubectl checksum:
```
curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
```
Comprobamos
```
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
```
![](img/kubernete-01.png)

Ahora ya podemos instalar
```
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```
Para asegurarnos de que tenemos una versión actualizada.
```
kubectl version --client=true --output=yaml
```
Ponemos `--cliente=true` porque si no se intentaría conectar a un clúster kubernetes y trataría de descargar la versión kubernetes del clúster. Para un formato más legible controlamos la salida con `--output=yaml|json]`

En *Windows*, con Docker Desktop podemos activar kubernetes.

En **Linux**, podemos instalar kind https://kind.sigs.k8s.io/

![](img/kind.png)

También podemos instalar minikube https://minikube.sigs.k8s.io/docs/ que instala todos los componentes de kubernetes en una MV y además tiene una serie de plugins para darle funcionalidades con una serie de paquetes precinfigurados.

![](img/minikube.png)

En esta guía utilizaré [DigitalOcean](https://m.do.co/c/98c9ca613f37), con 3 nodos kubernetes de los baratitos.

![](img/DigitalOcean.png)

Para conectar kubectl con el cluster de kubernetes se debe desarcargar el fichero *kubeconfig* que es donde están declarados los contextos de kubernetes. Es una combinación de la url del servidor con las credenciales de lo que se haya instalado. 

![](img/kubeconfig.png)

*Con minikube no tendríamos que hacer nada porque se configura automáticamente.*

Se exporta en una variable de entorno
```
export KUBECONFIG=~/Downloads/k8s-1-20..............
```
Y podremos comprobar los nodos con
```
kubectl get nodes
```

![](img/get-nodes.png)

Para verlo con colores se puede instalar el plugin de kubectl que se llama *[kubecolors](https://github.com/hidetatz/kubecolor)*

*En ubuntu:*
```
wget  http://archive.ubuntu.com/ubuntu/pool/universe/k/kubecolor/kubecolor_0.0.20-1_amd64.deb
```
```
sudo dpkg -i kubecolor_0.0.20-1_amd64.deb
```
```
sudo apt update && sudo apt install kubecolor -y

```
```
kubecolor --context=[tu_contexto] get pods -o json
```
```
kubecolor --context=do-ams3-k8s-1-24-4-do-0-ams3-1663572528096 get pods -o json
```
```
alias kubectl="kubecolor"
```
![](img/get-nodes-color.png)


Se puede ver la ayuda del cliente con 
```
kubectl --help
```
![](img/ayuda.png)

*Trascripción traducida:*
```
kubectl controla el gestor de clústeres de Kubernetes.

Encontrará más información en: https://kubernetes.io/docs/reference/kubectl/

Comandos básicos (principiante):
  create          Crear un recurso desde un archivo o desde stdin
  expose          Tomar un controlador de replicación, servicio, despliegue o pod y exponerlo como un nuevo servicio Kubernetes
  run             Ejecutar una imagen particular en el cluster
  set             Establecer características específicas en los objetos

Comandos básicos (Intermedio):
  explain         Obtener la documentación de un recurso
  get             Mostrar uno o varios recursos
  edit            Editar un recurso en el servidor
  delete          Eliminar recursos por nombres de archivo, stdin, recursos y nombres, o por recursos y selector de etiqueta

Comandos de despliegue:
  rollout         Gestionar el despliegue de un recurso
  scale           Establecer un nuevo tamaño para un despliegue, conjunto de réplicas o controlador de replicación
  autoscale       Escala automáticamente un despliegue, un conjunto de réplicas, un conjunto con estado o un controlador de replicación

Comandos de gestión del clúster:
  certificate     Modificar los recursos del certificado.
  cluster-info    Mostrar información del cluster
  top             Mostrar el uso de recursos (CPU/memoria)
  cordon          Marcar un nodo como no programable
  uncordon        Marcar el nodo como programable
  drain           Drenar el nodo en preparación para el mantenimiento
  taint           Actualizar los taints de uno o más nodos

Comandos de solución de problemas y depuración:
  describe        Mostrar los detalles de un recurso específico o de un grupo de recursos
  logs            Imprimir los registros de un contenedor en un pod
  attach          Adjuntar a un contenedor en ejecución
  exec            Ejecutar un comando en un contenedor
  port-forward    Reenviar uno o más puertos locales a un pod
  proxy           Ejecutar un proxy al servidor de la API de Kubernetes
  cp              Copiar archivos y directorios hacia y desde los contenedores
  auth            Inspeccionar la autorización
  debug           Crear sesiones de depuración para la resolución de problemas de cargas de trabajo y nodos

Comandos avanzados:
  diff            Comparar una versión en vivo contra una versión aplicada
  apply           Aplicar una configuración a un recurso por nombre de archivo o stdin
  patch           Actualizar los campos de un recurso
  replace         Reemplazar un recurso por nombre de archivo o stdin
  wait            Experimental: Esperar una condición específica en uno o varios recursos
  kustomize       Construir un objetivo de kustomize a partir de un directorio o una URL.

Settings Commands:
  label           Actualizar las etiquetas de un recurso
  annotate        Actualiza las anotaciones de un recurso
  completion      Imprimir el código de finalización del shell para el shell especificado (bash, zsh, fish o powershell)

Otros Comandos:
  alpha           Comandos para funciones en alpha
  api-resources   Imprime los recursos de la API soportados en el servidor
  api-versions    Imprime las versiones de la API admitidas en el servidor, en forma de "grupo/versión"
  config          Modifica los archivos kubeconfig
  plugin          Proporciona utilidades para interactuar con los plugins
  version         Imprime la información de la versión del cliente y del servidor

Uso:
  kubectl [flags] [options]

Utilice "kubectl <command> --help" para obtener más información sobre un determinado comando.
Utilice "kubectl options" para obtener una lista de opciones globales de la línea de comandos (se aplica a todos los comandos).
```

Una herramienta gráfica para kubectl es *[lens](https://k8slens.dev/)*., Muestra los contenedores de una manera clara y también tiene gráficas (memoria, CPU, etc).

Para mostrar los contextos que están en el fichero kubeconfig o el archivo de configuración de kubectl
```
kubectl config get-contexts
```

Para mostrar los namespaces:
```
kubectl get ns
```
Con el clúster recien creado aparecerán los namespaces por defecto que vienen con cualquier clúster.

![](img/get-ns.png)

Para ver los pods que están corriendo
```
kubectl -n kube-system get pods
```
![](img/kube-system-n-get-pods.png)

`kube-system` es un namespace que utiliza kubernetes para correr los pods de sistema.

Tomando un ejemplo, `do-node-agent-9rt5c` es un agente que corre DigitalOcean en sus nodos para hacer algún tipo de recolección de datos o monitoreo. El final alfanumérico es porque ha sido generado por el template de pods *deployment*, todos los pods tendrá ese hash en el nombre.

La segunda columna indica el número de pods activos y los que existen. El estado, está claro, después están las columnas de los reinicios efectuados y la de el tiempo que lleva arrancado.

Con la opción `-o wide` mostrará un poco más de información.

![](img/kube-system-n-get-pods-o-wide.png)

Vamos a probar lo que dicen de kubernetes de que si se borra un pod se creará uno nuevo. 
```
kubectl -n kube-system delete pod do-node-agent-9rt5c
```
Inmediatamente muestro los pods y se puede ver como lo está creando de nuevo.
![](img/delete-pod-agent.png)

El pod es nuevo, tiene otro hash. Así que esto asegura que siempre estén el mismo número de pods.

---
## Manifiestos de Pelado Nerd
### Manifiesto de POD
Ahora utilizaremos un manifiesto de un pod del [pelado Nerd](https://github.com/pablokbs/peladonerd/tree/master/kubernetes/35) llamado `01-pod.ymal`

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:alpine

```
Tendremos:
- La *versión de la API* de este recurso de kubernetes. Se debe extraer de la documentación de kubernetes.
- El *recurso*
- En *metadata* podemos poner algunos valores que identifiquen nuestro pod. Siempre tiene que tener un `name`.
- En las *especificaciones* declaramos los contenedores que corren. En este caso ponemos el nombre y la imagén en concreto.

Para aplicar este manifiesto
```
kubectl apply -f 01-pod.yaml
```
Si no especificamos un namespaces lo aplica en el que tenemos por defecto. Podremos mostrar que ya está corriendo.

![](img/apply-nginx.png)

Ahora, para correr un comando dentro del pod usaremos este comando
```
kubectl exec -it nginx -- sh
```
Con la misma opción de docker, `-it` nos permite que sea interactivo.

![](img/exec-it-sh.png)

Para salir es con `CTRL+d`.

Borrar un pod
```
kubectl delete pod nginx
```
Como no se creo la orden para mantener siempre un pod, el pod desapareció. 

##Manifiestos del Pelado Nerd

Otro manifiesto de un pod del [pelado Nerd](https://github.com/pablokbs/peladonerd/tree/master/kubernetes/35) llamado `02-pod.ymal` es lo mismo pero con más opciones.

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    env:
    - name: MI_VARIABLE
      value: "pelado"
    - name: MI_OTRA_VARIABLE
      value: "pelade"
    - name: DD_AGENT_HOST
      valueFrom:
        fieldRef:
          fieldPath: status.hostIP
    resources:
      requests:
        memory: "64Mi"
        cpu: "200m" # son milicores, cada core tiene 1000 milicores
      limits:
        memory: "128Mi"
        cpu: "500m" 
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:
      tcpSocket:
        port: 80
      initialDelaySeconds: 15
      periodSeconds: 20
    ports:
    - containerPort: 80
```

En este tenemos más especificaciones:
- *Variables de entorno* como clave-valor. Con `DD_AGENT_HOST` podemos pasar el valor de otro sitio gracias a *[Downward](https://kubernetes.io/docs/concepts/workloads/pods/downward-api/)* de kubernetes, que son valores que se pueden heredar, en este caso indicamos la ip del host donde va a correr este pod: `status.hostIP`.
- Se indican los recursos que se garantizan por contenedor con `requests` y los limites con `limits`. Los límites provocan que el kernel de Linux haga CPU Throttling, es decir, ahorcará el proceso hasta que use la velocidad límite y si no lo matará y esto hará que se cree otro pod.
- *ReadinessProbe* es una forma de explicarle a kubernetes de que el pod está preparado para recibir tráfico. Kubernetes comprueba la raíz esperando un status code 200.
- *livenessProbe*  es una forma de explicarle a kubernetes de que el pod está vivo y no quieres que lo mate. Kubernetes comprueba el socker del puerto 80 de que está vivo.
- Por último tenemos el *puerto* que queremos poner.

Vamos a correrlo
```
kubectl apply -f pelado_nerd_pruebas/kubernetes/35/02-pod.yaml
```
Podemos ver el estado del pod con
```
 kubectl get pod nginx
```
Y además veremos el yaml si le añadimos la opción `-o yaml` con todas las variables y parámetros por defecto que le ha añadido kubernetes.
```
 kubectl get pod nginx -o yaml
```
### Manifiesto de Deployment
Pero para trabajar con kubernetes, no deberíamos levantar el recurso mínimo como son los pods, es mejor aprocechar la capacidad de orquestación de la herramienta y levantar deployment. Lo encontraremos en otro manifiesto de un deployment del [pelado Nerd](https://github.com/pablokbs/peladonerd/tree/master/kubernetes/35) llamado `04-deployment.yaml`.
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        env:
        - name: MI_VARIABLE
          value: "pelado"
        - name: MI_OTRA_VARIABLE
          value: "pelade"
        - name: DD_AGENT_HOST
          valueFrom:
            fieldRef:
              fieldPath: status.hostIP
        resources:
          requests:
            memory: "64Mi"
            cpu: "200m"
          limits:
            memory: "128Mi"
            cpu: "500m"
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
        livenessProbe:
          tcpSocket:
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 20
        ports:
        - containerPort: 80

```

Es muy parecido al yaml de los pods pero especificando como una plantilla de pods en el `spec` dentro del `spec`.

Las réplicas son el número de pods que queremos dentro del deployment.

Aplicamos el manifiesto
```
kubectl apply -f 04-deployment.yaml
```
Y veremos como ha desplegado las 2 réplicas que se indica en el manifiesto.

![](img/apply-deployment.png)

Ahora sí que si borramos uno de los pods, como en el manifiesto indicamos 2 réplicas, kubernetes lo creará de nuevo.

![](img/delete-pod-nginx.png)

Para borrar el deployment utilizaremos el mismo yaml con el que lo corrimos.
```
kubectl delete -f 04-deployment.yaml
```
### Manifiesto de daemonset
`daemonset` es otra forma de deployment un pod pero en este caso será un pod en cada uno de los nodos existente. No indicas las réplicas por eso. Sirve, por ejemplo, para despliegues de servicios de monitoreo. De nuevo, vamos a un manifiesto de un daemonset del [pelado Nerd](https://github.com/pablokbs/peladonerd/tree/master/kubernetes/35) llamado `03-daemonset.yaml`.
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        env:
        - name: MI_VARIABLE
          value: "pelado"
        - name: MI_OTRA_VARIABLE
          value: "pelade"
        - name: DD_AGENT_HOST
          valueFrom:
            fieldRef:
              fieldPath: status.hostIP
        resources:
          requests:
            memory: "64Mi"
            cpu: "200m"
          limits:
            memory: "128Mi"
            cpu: "500m"
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
        livenessProbe:
          tcpSocket:
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 20
        ports:
        - containerPort: 80
```
Veremos como se han desplegado 3 pods, uno por nodo.

![](img/apply-daemonset.png)

Borramos de nuevo con el fichero
```
kubectl delete -f pelado_nerd_pruebas/kubernetes/35/03-daemonset.yaml
```

> Continuará....











---
## Cheatsheet kubernetes

### Visualizar información de los recursos
#### Nodes
```
kubectl get no
```
```
kubectl get no -o wide
```
```
kubectl describe no
```
```
kubectl get no -o yaml
```
```
kubectl get node –select or =[ label _name]
```
```
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type==”ExternalIP”)].address}’
```
```
kubectl top node [node_name]
```
#### ods
```
kubectl get po
```
```
kubectl get po -o wide
```
```
kubectl describe po
```
```
kubectl get po –show-labels
```
```
kubectl get po -l app=nginx
```
```
kubectl get po -o yaml
```
```
kubect l get pod [ pod_name] -o yaml –export
```
```
kubect l get pod [pod_name] -o yaml –export > nameoffile.yaml
```
```
kubectl get pods –field-selector status.phase=Running
```
#### Namespaces
```
kubectl get ns
```
```
kubectl get ns -o yaml
```
```
kubectl describe ns
```
#### Deployments
```
kubectl get deploy
```
```
kubectl describe deploy
```
```
kubectl get deploy -o wide
```
```
kubectl get deploy -o yam
```
#### Services
```
kubectl get svc
```
```
kubectl describe svc
```
```
kubectl get svc -o wide
```
```
kubectl get svc -o yaml
```
```
kubectl get svc –show-labels
```
#### DaemonSets
```
kubectl get ds
```
```
kubectl get ds –all-namespaces
```
```
kubectl describe ds [daemonset _name] -n [namespace_name]
```
```
kubectl get ds [ds_name] -n [ns_name] -o yaml
```
#### Events
```
kubectl get events
```
```
kubectl get events -n kube-system
```
```
kubectl get events -w
```
#### Logs
```
kubectl logs [pod_name]
```
```
kubectl logs –since=1h [pod_name]
```
```
kubectl logs –tail =20 [pod_name]
```
```
kubectl logs -f -c [container_name] [pod_name]
```
```
kubectl logs [pod_name] > pod.log
```
#### Service Accounts
```
kubectl get sa
```
```
kubectl get sa -o yaml
```
```
kubectl get serviceaccounts default -o yaml > ./sa.yaml
```
```
kubectl replace serviceaccount default -f. /sa.yaml
```
#### ReplicaSets
```
kubectl get rs
```
```
kubectl describe rs
```
```
kubectl get rs -o wide
```
```
kubectl get rs -o yaml
```
#### Roles
```
kubectl get roles –all-namespaces
```
```
kubectl get roles –all-namespaces -o yaml
```
#### Secrets
```
kubectl get secrets
```
```
kubectl get secrets –all-namespaces
```
```
kubectl get secrets -o yaml
```
#### ConfigMaps
```
kubectl get cm
```
```
kubectl get cm –all-namespaces
```
```
kubectl get cm –all-namespaces -o yaml
```
#### Ingress
```
kubectl get ing
```
```
kubectl get ing –all-namespaces
```
#### PersistentVolume
```
kubectl get pv
```
```
kubectl describe pv
```
#### PersistentVolumeClaim
```
kubectl get pvc
```
```
kubectl describe pvc
```
#### StorageClass
```
kubectl get sc
```
```
kubectl get sc -o yaml
```
#### MultipleResources
```
kubectl get svc, po
```
```
kubectl get deploy, no
```
```
kubectl get all
```
```
kubectl get all –all-namespaces
```
### Modificar atributos de los recursos
#### Taint
```
kubectl taint [node_name] [taint _name]
```
#### Labels
```
kubectl label [node_name] disktype=ssd
```
```
kubrectl label [pod_name] env=prod
```
#### Cordon/Uncordon
```
kubectl cordon [node_name]
```
```
kubectl uncordon [node_name]
```
#### Drain
```
kubectl drain [node_name]
```
#### Nodes/Pods
```
kubectl delete node [node_name]
```
```
kubectl delete pod [pod_name]
```
```
kubectl edit node [node_name]
```
```
kubectl edit pod [pod_name]
```
#### Deployments/Namespaces
```
kubectl edit deploy [deploy_name]
```
```
kubectl delete deploy [deploy_name]
```
```
kubectl expose deploy [depl oy_name] –port=80 –type=NodePort
```
```
kubectl scale deploy [deploy_name] –replicas=5
```
```
kubectl delete ns
```
```
kubectl edit ns [ns_name]
```
#### Services
```
kubectl edit svc [svc_name]
```
```
kubectl delete svc [svc_name]
```
#### DaemonSets
```
kubectl edit ds [ds_name] -n kube-system
```
```
kubectl delete ds [ds_name]
```
#### ServiceAccounts
```
kubectl edit sa [sa_name]
```
```
kubectl delete sa [sa_name]
```
#### Annotate
```
kubectl annotate po [pod_name] [annotation]
```
```
kubectl annotate no [node_name]
```
### Añadir recursos
#### Crear Pod
```
kubectl create -f [name_of _file]
```
```
kubectl apply -f [name_of _file]
```
```
kubectl run [pod_name] –image=ngi nx –restart=Never
```
```
kubectl run [ pod_name] –generator =run-pod/v1 –image=nginx
```
```
kubectl run [ pod_name] –image=nginx –restart=Never
```
#### Crear un Service
```
kubectl create svc nodeport [svc_name] –tcp=8080:80
```
#### Crear Deployment
```
kubectl create -f [name_of _file]
```
```
kubectl apply -f [name_of _file]
```
```
kubectl create deploy [deploy_name] –image=ngi nx
```
#### Interactive Pod
```
kubectl run [pod_name] –image=busybox –rm -it –restart=Never — sh
```
#### Salida de YAMLto en un fichero
```
kubectl create deploy [deploy_name] –image=ngi nx –dry-run -o yaml > deploy.yaml
```
```
kubectl get po [pod_name] -o yaml –export > pod. yaml
```
#### Ayuda
```
kubectl -h
```
```
kubectl create -h
```
```
kubectl run -h
```
```
kubectl explain deploy.spec
```
### Solicitaciones
#### Llamar a la API
```
kubectl get –raw /apis/metrics.k8s.io/
```
#### Información del Cluster
```
kubectl config
```
```
kubectl cluster -info
```
```
kubectl get componentstatuses
```

![](img/kubernetes-cheat-sheet.png)




---
## Ejemplo de un YAML para un pod básico de busybox
```
apiVersion: v1kind: Podmetadata:name: busyboxspec:containers:- image: busybox:1.28.4command:- sleep- "3600"name: busyboxrestartPolicy: Always
```
Crear un *pod*
```
kubectl create -f busybox.yaml
```
Crear un *deployment*
```
kubectl run nginx --image=nginx
```
Crear un *service* a partir del *deployment* anterior
```
kubectl expose deployment nginx --port=80 --type=NodePort
```
Aquí está el *YAML* para un *volumen persistente* simple usando el almacenamiento local del nodo:
```
apiVersion: v1kind: PersistentVolumemetadata:name: data-pvnamespace: webspec:storageClassName: local-storagecapacity:storage: 1GiaccessModes:- ReadWriteOncehostPath:path: /mnt/data
```
Crear un *volumen persistente*
```
kubectl apply -f my-pv.yaml
```
Aquí está el *YAML* para un *ConfigMap* simple
```
apiVersion: v1kind: ConfigMapmetadata:name: my-config-mapdata:myKey: myValueanotherKey: anotherValue
```
Crear el *ConfigMap*
```
kubectl apply -f configmap.yaml
```
Aquí está el *YAML* para los *secret*:
```
apiVersion: v1kind: Secretmetadata:name: my-secretstringData:myKey: myPassword
```
Crear el *secret*
```
kubectl apply -f secret.yaml
```
Aquí está el *YAML* para una *cuenta de servicio*
```
apiVersion: v1kind: ServiceAccountmetadata:name: acrnamespace: defaultsecrets:- name: acr
```
Crear el *service account*
```
kubectl apply -f serviceaccount.yaml
```

## Agradecimientos

Esta guía ha sido creada a partir de multitud de tutoriales que he hecho, son mis apuntes personales. Pero quiero hacer una especial mención a [Pelado Nerd](https://www.youtube.com/c/PeladoNerd), espero que la guía sea como el Pelado manda.
