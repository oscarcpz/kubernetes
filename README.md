# Primeros pasos

## Conceptos básicos

### Pod

Podría ser la máquina virtual a día de hoy. Comparte la ip, cpu, recursos. Pero en vez de contener procesos, contiene "contenedores". **Un pod puede tener más de un contenedor**

Tiene ip dinámica

### Replicaset/Deployment

Comprueba periódicamente el estado de nuestros pods. Cuando encuentre un pod en un estado que no está correcto, lo corrige.

### Servicio

Agrupa pods por etiquetas, les da una ip estática y una DNS.

Es la forma de poder comunicar con los pods.

### Persistent Volumes

Para poder hacer persistentes los datos en los pods.

### Cluster

#### Nodos

##### Master

Gestiona el cluster

* api
* etcd - base de datos del cluster
* scheduler
* controllers

##### Worker nodes

Ejecuta los contenedores

## Instalar Kubernetes

Lo más recomendable es seguir la [guía de instalación](https://kubernetes.io/es/docs/tasks/tools/install-kubectl/)

Una vez que tengamos Kubernetes instalado se puede comprobar con el siguiente comando

~~~
$ kubectl version --client
~~~

## Instalar Minicube

Para una instalación en local lo mejor es hacer uso de [Minicube](https://minikube.sigs.k8s.io/docs/start/)

## Algunos comandos

### Inicio

* `$ minikube start` - levantar el cluster

### Comandos de utilidad

* `$ kubectl get pod -A` - muestra todos los pods (equivale a _docker ps -a_)
* `$ kubectl get all` - muestra todo lo que tenemos en el cluster
* `$ minikube dashboard` - arranca un dashboard que muestra información del cluster

### Desplegar aplicaciones en línea de comandos

Sugerencia: para ver cómo funciona el cluster, los siguientes comandos ejecutarlos en paralelo con el siguiente comando en otra venta `$ watch kubectl get pods`

1. `$ kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.4` - creamos el pod (equivale a _docker run_)
* `$ kubectl get deployments` - muestra información de los contenedores desplegados
* `$ kubectl describe pod <nombre_pod>` - devuelve información del pod, como la ip asignada del worker node
2. `$ kubectl expose deployment hello-minikube --type=LoadBalancer --port=8080` - exponemos el puerto al que queremos dirigirlo. Podremos acceder por el nombre del servicio o por la ip __CLUSTER-IP__. Sirve para poder acceder a los pods sin necesidad de conocer la ip dinámica que se les asigna. Ojo al tipo de acceso que se ha seleccionado: **LoadBalancer**
3. `$ kubectl get services hello-minikube` - comprobamos que tenemos el servicio.
* `$ kubectl get services`
~~~
NAME             TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hello-minikube   NodePort   10.96.150.162   <none>        8080:30172/TCP   27m
~~~
* Podemos dejar a **minikube** que lance un navegador por nosotros `$ minikube service hello-minikube`
* O buscar la ip a la que nos debemos conectar `$ minikube ip`

4. `$ kubectl exec -ti <nombre_app> /bin/bash` - accede a la consola dentro del pod
5. `$ kubectl logs <nombre_pod>` - para ver los logs
6. `$ kubectl scale deployment <nombre_app> --replicas=2` - para escalar la aplicación (se puede ver bien con una ventana paralela ejecutando **watch**). Gracias al servicio, se hará transparente el acceso mediante **LoadBalancer**
* Otra forma de escalar sería editando el fichero de despliegue `$ kubectl edit deployment <nombre_app>`. Abre el fichero para que se pueda editar. Modificados: spec.replicas y ponemos el número que queramos. Cuando se guarde, automáticamente se cambiará el escalado.
7. `$ kubectl set image deployment/<nombre_app> <nombre_nueva_imagen>` - se cambia la versión del contenedor con la nueva imagen.
8. `$ kubectl delete all --all` - borrar todo lo que tenga el cluster. **Solo usar en desarrollo**

### Desplegar aplicaciones por fichero yml

#### Básico
~~~
apiVersion: v1
kind: Pod
metadata:
  name: miapp
spec:
  containers:
  - name: miapp
    image: <nombre_imagen>
~~~

`$ kubectl apply -f <fichero.yml>`
`$ kubectl delete pod miapp` - borra el pod pero no levanta otro. Esto es debido a que no tenemos un **deployment**

#### Deployment

~~~
apiVersion: v1
kind: Deployment
metadata:
  name: miapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: miapp
  template:
    labels:
      app: miapp
      env: dev
  spec:
    containers: # plantilla que se aplicará a cada pod que se cree indicado en el "replicas"
      - name: miapp
        image: <nombre_imagen>
        imagePullPolicy: Always
        ports:
          - containerPort: 8000
~~~

* `$ kubectl apply -f <fichero_deployment.yml>`

* `$ kubectl delete pod <nombre_pod>` - Si ahora matamos un pod, inmediatamente creará uno nuevo
* `$ kubectl describe deployment <nombre_deployment>` - muestra mucha más información

#### Servicio

~~~
apiVersion: v1
kind: Service
metadata:
  name: the-service
spec:
  selector:
    app: miapp # para indicar a las aplicaciones que quieres acceder
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer
~~~

*`$ kubectl apply -f <fichero_servicio.yml>`

* `$ kubectl get services` - nos mostrará la ip que podremos usar para conectar

### Ejemplo más real

~~~
apiVersion: v1
kind: Deployment
metadata:
  name: mypython-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mypython
  template:
    labels:
      app: mypython
  spec:
    containers:
      - name: mypython
        image: <nombre_imagen_python>
        imagePullPolicy: Always
        ports:
          - containerPort: 8000
~~~

`$ kubectl apply -f <fichero_despliegue_python.yml>`

---

~~~
apiVersion: v1
kind: Deployment
metadata:
  name: mygo-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mygo
  template:
    labels:
      app: mygo
  spec:
    containers:
      - name: mygo
        image: <nombre_imagen_go>
        imagePullPolicy: Always
        ports:
          - containerPort: 8000
~~~

`$ kubectl apply -f <fichero_despliegue_go.yml>`

---

~~~
apiVersion: v1
kind: Deployment
metadata:
  name: mynode-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mynode
  template:
    labels:
      app: mynode
  spec:
    containers:
      - name: mynode
        image: <nombre_imagen_node>
        imagePullPolicy: Always
        ports:
          - containerPort: 8000
~~~

`$ kubectl apply -f <fichero_despliegue_node.yml>`

---

~~~
apiVersion: v1
kind: Service
metadata:
  name: my-service
  labels:
    app: mystuff
spec:
  ports:
  - name: http
    port: 8000
  selector:
    inservice: mypods # se aplicará a todos los pods que tengan la etiqueta "mypods"
  type: LoadBalancer
~~~

* `$ kubectl apply -f <fichero_servicio.yml>` - ahora mismo, con la configuración creada en los ficheros anteriores, el servicio no estará redirigiendo a ningún pod porque ninguno tiene la etiqueta "inservice=mypods"
* `$ kubectl describe service my-service` - se puede ver que el campo "endpoints" está vacío

Para que funcione, se tendrá que añadir la etiqueta a algún pod:
* `$ kubectl label pod -l app=mygo inservice=mypods`

* `$ kubectl describe service my-service` - ahora sí mostrará una ip en el endpoint

Si se aplica la etiqueta a los tres tipos de pods, cuando se haga un curl a la ip estática del servicio, el LoadBalancer irá cambiando de pod

### Administrar el cluster con minikube

* `$ minikube status` - Conocer el estado del cluster

~~~
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
~~~

* `$ minikube stop` - Parar el cluster

## Tipos de accesos

* **NodePort** - es la forma básica de acceso. Consiste en redirigir el tráfico directamente al servicio por un puerto determinado.
* **LoadBalancer** -

# Stern

Aplicación que permite mostrar todos los logs de todos los pods al mismo tiempo

`$ stern miapp<regex>` - para mostrar todos los logs del cluster cuyo nombre cumpla con la expresión regular

# Referencias

* [Kubernetes para aquellos que manejan docker](https://kubernetes.io/docs/reference/kubectl/docker-cli-to-kubectl/)
* [Entorno para jugar con Kubernetes](https://labs.play-with-k8s.com)
* [Stern](https://github.com/wercker/stern)
