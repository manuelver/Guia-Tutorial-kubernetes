# Helm: Control de despliegues en Kubernetes

**Índice**
- [Helm: Control de despliegues en Kubernetes](#helm-control-de-despliegues-en-kubernetes)
  - [¿Qué es Helm?](#qué-es-helm)
  - [Instalación de Helm](#instalación-de-helm)
  - [Opciones del comando helm](#opciones-del-comando-helm)
  - [Crear una release](#crear-una-release)
  - [Apuntes más completos](#apuntes-más-completos)

## ¿Qué es Helm?

[Helm](https://helm.sh/) es uno de los proyectos más interesantes dentro de la comunidad de Kubernetes.

La idea de Helm es controlar un **despliegue** (lo llaman release) de tal forma que:

- Usando un solo conjunto de valores (generalmente expresado en YAML):
  - Todos los artefactos que lo componen (deploys, pods, configmaps, services...) tienen reflejados los valores de configuración correctos.
  - Están declarados correctamente en el clúster K8s.
  - Ante un cambio de valores, se reconfiguran los artefactos correspondientes.
- El release, con un solo comando, puede:
  - Listarse
  - Detenerse
  - Actualizarse
  - Reconfigurarse
- Los releases parten de planes o [charts](https://github.com/helm/charts), es decir, repositorios con el código necesario para lanzar una aplicación en Kubernetes:
  - Se encuentran en repositorios públicos.
  - Los hay de todos los tipos (mysql, mongo, Wordpress...).
  - Se pueden descargar y utilizar o ampliar como queramos.

## Instalación de Helm

En nuestro ubuntu es muy sencillo. 

- Con snap:

```shell
snap install helm --classic
```

- Con apt:

```shell
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null

sudo apt-get install apt-transport-https --yes

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list

sudo apt-get update

sudo apt-get install helm
```

 **¡Nota! El binario está en /snap/bin/helm, si no está en la ruta, puede incluirlo o crear un enlace simbólico, por ejemplo, a /usr/sbin/helm.*

En versiones anteriores era necesario inicializar helm para empezar a trabajar con él, pero en la versión 3 ya no es así. Si estamos usando una versión anterior, tendremos que lanzar el siguiente comando:

```shell
helm init
```

**¡Nota! Si dice que las versiones son incompatibles, haga `helm init --upgrade`.*

**¡Nota 2! helm puede tardar un minuto en ponerse en marcha. Espere hasta que pueda interactuar con él.*

Ahora, podemos crear un release.

## Opciones del comando helm

  helm [comando]

  |  comando   | Descripción                                                                                             |
  | :--------: | ------------------------------------------------------------------------------------------------------- |
  | completion | Generar scripts de autocompletado para el shell especificado                                            |
  |   create   | Crear un nuevo chart con el nombre indicado                                                             |
  | dependency | Gestionar las dependencias de un chart                                                                  |
  |    env     | Información del entorno de cliente                                                                      |
  |    get     | Descargar información ampliada de la release nombrada                                                   |
  |    help    | Ayuda sobre cualquier comando                                                                           |
  |  history   | Obtener el historial de release                                                                         |
  |  install   | Instalar una chart                                                                                      |
  |    lint    | Examinar posibles incidencias de una chart                                                              |
  |    list    | listar releases                                                                                         |
  |  package   | Empaquetar un directorio chart en un fichero chart                                                      |
  |   plugin   | Instalar (install), listar (list) o desinstalar (uninstall) plugins Helm                                |
  |    pull    | Descargar una chart de un repositorio y (opcional) desempaquetarlo en directorio local                  |
  |    repo    | Añadir (add), listar (list), borrar (remove), actualizar (update), e indexar (index) repositorios chart |
  |  rollback  | roll back un release a la revisión anterior                                                             |
  |   search   | Buscar keyword en las charts                                                                            |
  |    show    | Mostrar información de una chart                                                                        |
  |   status   | Mostrar el estado de una release nombrada                                                               |
  |  template  | Representar localmente templates                                                                        |
  |    test    | Ejecutar pruebas a una release                                                                          |
  | uninstall  | desinstalar una release                                                                                 |
  |  upgrade   | Actualizar una release                                                                                  |
  |   verify   | verificar si una chart en una path ha sido firmada y validada                                           |
  |  version   | Mostrar información de la version del cliente helm                                                      |


## Crear una release

Para crear un release, descargue el **chart** o el plano o pídale a helm que lo descargue él mismo.

Los charts están en repositorios.

```shell
# listamos os repos
helm repo list

NAME            URL
local           http://127.0.0.1:8879/charts

# Engadimos o repo de bitnami, un dos máis utilizados
 helm repo add bitnami https://charts.bitnami.com

"bitnami" has been added to your repositories

# Listamos os charts dispoñibles
helm search repo bitnami
NAME                                    CHART VERSION   APP VERSION                     
bitnami/airflow                                 13.0.4          2.3.3           Apache Airflow is a tool to express and execute...
bitnami/apache                                  9.1.18          2.4.54          Apache HTTP Server is an open-source HTTP serve...
bitnami/argo-cd                                 4.0.6           2.4.8           Argo CD is a continuous delivery tool for Kuber...
```

Podemos, por ejemplo, instalar un mariadb:

```shell
helm install bbdd bitnami/mariadb
```
Para ver el release

```shell
helm list

NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
bbdd    default         1               2022-08-11 10:12:54.717064554 -0400 EDT deployed        mariadb-11.1.7  10.6.8      
```

Si exploramos nuestro clúster:

```shell
kubectl get pods

NAME             READY   STATUS    RESTARTS   AGE
bbdd-mariadb-0   1/1     Running   0          98s

```

También creó el servicio vinculado al pod, un secreto para password y un configmap.

Es decir, ¡tenemos un MariaDB instalado con las mejores prácticas de la industria para Kubernetes!

Para configurarlo hay que ir a [artifacthub](https://artifacthub.io/packages/helm/bitnami/mariadb), donde nos indican los valores a modificar para gestionar nuestra instalación.

Ponga esos valores en un archivo yaml:

```yaml
# values.yaml
db:
  username: paco
  password: segredo
  database: test
```

Ahora estamos relanzando nuestro release:

```shell
# borramos a release
helm uninstall bbdd

# relanzamos cos values
helm install maria -f values.yaml bitnami/mariadb
```

Y tendríamos el deployment con una base de datos creada y un usuario vinculado a ella.

## Apuntes más completos

Este documento contiene los apuntes tomados en el curso «[Helm 3: Despliega aplicaciones en Kubernetes](https://www.udemy.com/course/helm-3-despliega-aplicaciones-en-kubernetes/)» impartido por [Apasoft Training](https://www.linkedin.com/in/apasoft-training-b38b36134/) en diciembre de 2022. El curso udemy consta de 5 horas aproximadamente de vídeo-tutoriales. Las prácticas aquí contenidas tuvieron una duración de alrededor de unas 12 horas.

_En breve subiré un pdf con los apuntes de un curso bastante completo_

<br> <br>

---

Puedes seguir con la guía [06 K9s: Otro estilo de CLI para k8s](06-k9s.md).

Todas las guías:

- [01 Instalación kubctl](01-kubectl.md) 
- [02 Clústers](02-clusters.md) 
- [03 manifiestos](03-manifiestos.md) 
- [04 Cheatsheet kubernetes](04-cheatsheet.md) 
- [05 Helm: Control de despliegues en Kubernetes](05-helm.md) 
- [06 K9s: Otro estilo de CLI para k8s](06-k9s.md)
