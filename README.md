# Guía-Tutorial Kubernetes

## Guías

[01 Instalación kubctl](guias/01-kubectl.md) --> [02 Clústers](guias/02-clusters.md) --> [03 manifiestos](guias/03-manifiestos.md) --> [04 Cheatsheet kubernetes](guias/04-cheatsheet.md) --> [05 Helm: Control de despliegues en Kubernetes](guias/05-helm.md)

**Índice** 📎

- [Guía-Tutorial Kubernetes](#guía-tutorial-kubernetes)
  - [Guías](#guías)
  - [¿Qué es Kubernetes?](#qué-es-kubernetes)
  - [Componentes](#componentes)
  - [Recursos Kubernetes](#recursos-kubernetes)
  - [Ejemplo de ficheros YAML](#ejemplo-de-ficheros-yaml)
- [Agradecimientos 🎁](#agradecimientos-)

## ¿Qué es Kubernetes?

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
## Ejemplo de ficheros YAML 

Utilizaremos un pod con busybox.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  containers:
  - image: busybox:1.28.4
    command:
      - sleep
      - "3600"
    name: busybox
  restartPolicy: Always
```
Crear un *pod*
```shell
kubectl create -f busybox.yaml
```
Crear un *deployment*
```shell
kubectl run nginx --image=nginx
```
Crear un *service* a partir del *deployment* anterior
```shell
kubectl expose deployment nginx --port=80 --type=NodePort
```
Para darle persistencia a los datos utilizaremos un *YAML* para un *volumen persistente* simple usando el almacenamiento local del nodo:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: data-pv
  namespace: web
spec:
  storageClassName: local-storage
    capacity:
      storage: 1Gi
    accessModes:
    - ReadWriteOnce
    hostPath:
      path: /mnt/data
```
Crear un *volumen persistente*.
```shell
kubectl apply -f my-pv.yaml
```
Podemos darle configuración con el *YAML ConfigMap*.
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config-map
data:
  myKey: 
    myValueanotherKey: anotherValue
```
Crear el *ConfigMap*
```shell
kubectl apply -f configmap.yaml
```
Guardaremos las contraseñas en un *YAML* tipo *secret*:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
  stringData:
    myKey: myPassword
```
Crear el *secret*
```shell
kubectl apply -f secret.yaml
```
Aquí está el *YAML* para una *cuenta de servicio*
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: acr
  namespace: default
secrets:
  - name: acr
```
Crear el *service account*
```shell
kubectl apply -f serviceaccount.yaml
```

---

<br><br>

# Agradecimientos 🎁

Esta guía ha sido creada a partir de multitud de tutoriales que he hecho, son mis apuntes personales. Pero quiero hacer unas menciones especiales a: 
- [**Pelado Nerd**](https://www.youtube.com/c/PeladoNerd). Espero que la guía sea como el Pelado manda.
- [**Prefapp**](https://prefapp.es/). De donde he folkeado el [tutorial de helm](https://github.com/prefapp/formacion/blob/master/cursos/kubernetes/03_configuracion/07_Helm.md).

<br>

[Manu](https://vergaracarmona.es) 😊