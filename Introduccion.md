# Kubernetes
## Introducción
### Aquitectura de microservicios:
**Aplicación monolítica**:
- Todos los componentes en el mismo nodo.
- Escalado vertical.
- Arquitectura muy sencilla.
- Suele utilizarse un solo servicio.

**Aplicación distribuida**:
- Idealmente un componente por nodo.
- Escalado horizontal.
- Arquitectura más compleja.
- Menos interferencias entre componentes.
- Mayor simplicidad en las actualizaciones.
- Diferentes enfoques no excluyentes: SOA, cloud native, microservicios...

**Microservicios**:
- Deriva del esquema SOA.
- Solución más pragmática.
- Servicios llevados a la mínima expresión (idealmente un proceso por nodo).
- Comunicación vía HTTP REST y colas de mensajes, con el protocolo gRPC.
- Comando con procesos ágiles de desarrollo, facilita enormemente la actualizaciones de versiones, llegando incluso a la entrega continua o despliegue continuo.
- Suele implementarse sobre contenedores.
- Aumenta de lantencia entre componentes.

## Usos y mejoras de docker
- Balanceador de carga entre contenedores.
- Conexión entre conectedores.
- Actualizaciones sin interrupción.
- Variación a demanda el número de réplicas.

Tiene capacidad para crear nuevos nodos si le hace falta, porque tiene driver compatibles con la nube. Esta característica se pierde en cloud físicos.

## Complementos
- Pods y servicios que proporcionan funcionalidad al clúster.
- Cluster DNS.
- Web UI.
- Container Resource Monitoring. Recoge las métricas de forma centralizada.
...

## Nodos
- kubelet: recibe las órdenes y se encarga de gestionar los contenedores.
- kube-proxy: premite la conexión a través de la red.
- docker/rkt/containerd...
...



## Instalación
~~~
paloma@coatlicue:~/kubernete$ curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
> && chmod +x minikube
~~~

Para tener disponible el comando minikube:
~~~
paloma@coatlicue:~/kubernete$ sudo cp minikube /usr/local/bin && rm minikube
~~~

Se indica la forma de virtualización, a veces te lo reconoce solo, aquí, en este caso, se indica que ninguno:
~~~
paloma@coatlicue:~/kubernete$ sudo cp minikube /usr/local/bin && rm minikube
~~~

Se descarga el cliente de kubernete:
~~~
paloma@coatlicue:~/kubernete$ curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
~~~

Se siguen las instrucciones de la [página oficial de kubernete](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
~~~
paloma@coatlicue:~/kubernete$ chmod +x ./kubectl
paloma@coatlicue:~/kubernete$ sudo mv ./kubectl /usr/local/bin/kubectl
paloma@coatlicue:~/kubernete$ kubectl version --client
Client Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.3", GitCommit:"06ad960bfd03b39c8310aaf92d1e7c12ce618213", GitTreeState:"clean", BuildDate:"2020-02-11T18:14:22Z", GoVersion:"go1.13.6", Compiler:"gc", Platform:"linux/amd64"}
~~~

Se inicia:
~~~
paloma@coatlicue:~/kubernete$  minikube start --vm-driver=virtualbox
~~~

## Crear un pod
Unidad más pequeña con los que podemos corresr contenedores. Un pod representa un conjunto de contenedores que comparte almacenamiento y una única IP. Los pod son efímeros. 

Ejemplo de un pod con dos contenedores: wordpress con php. 

Fichero de configuración pod.yaml
~~~
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: default
  labels:
    app: nginx
    service: web
spec:
  containers:
    - image: nginx:1.16
      name: nginx
      imagePullPolicy: Always
~~~

~~~
paloma@coatlicue:~/kubernete$ nano pod.yaml
paloma@coatlicue:~/kubernete$ kubectl create -f pod.yaml 
pod/nginx created
paloma@coatlicue:~/kubernete$ kubectl get pods
NAME    READY   STATUS              RESTARTS   AGE
nginx   0/1     ContainerCreating   0          7s
paloma@coatlicue:~/kubernete$ 
~~~

## Replicaset
Recurso de kubernetes que asegura que siempre se ejecute un número de réplicas de un pod determinado.

Fichero replicaset.yml:
~~~
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - image:  nginx
          name:  nginx
~~~

Se añaden:
~~~
paloma@coatlicue:~/kubernete$ kubectl create -f replicaSet.yaml 
replicaset.apps/nginx created
paloma@coatlicue:~/kubernete$ kubectl get pods
NAME          READY   STATUS    RESTARTS   AGE
nginx         1/1     Running   0          31m
nginx-6p5j4   1/1     Running   0          4m26s
~~~

Escalar pods:
~~~
paloma@coatlicue:~/kubernete$ kubectl scale rs nginx --replicas=5
replicaset.apps/nginx scaled
paloma@coatlicue:~/kubernete$ kubectl get pods -o wide
NAME          READY   STATUS              RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
nginx         1/1     Running             0          34m     172.17.0.4   minikube   <none>           <none>
nginx-6p5j4   1/1     Running             0          6m43s   172.17.0.5   minikube   <none>           <none>
nginx-fd7t2   0/1     ContainerCreating   0          4s      <none>       minikube   <none>           <none>
nginx-mxdbx   0/1     ContainerCreating   0          4s      <none>       minikube   <none>           <none>
nginx-skb4n   0/1     ContainerCreating   0          4s      <none>       minikube   <none>           <none>
~~~

## Deployment
Es la unidad de más alto nivel que podemos gestionar en kubernetes. Nos permite definir diferentes funciones:
- Control de réplicas
- Escabilidad de pods
- Actualizaciones continuas
- Despliegues automáticos
- Rollback a versiones anteriores

Sin fichero yaml puedes hacer un deployment por defecto:
~~~
kubectl create deployment nginx --image=nginx
~~~

deployment.yaml:
~~~
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: default
  labels:
    app: nginx
spec:
  revisionHistoryLimit: 2
  strategy:
    type: RollingUpdate # hay dos estrategias
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - name: http
          containerPort: 80
~~~
Las dos estrategias:
- recreate: elimina los Pods antiguos y crea los nuevos.
- RollingUpdate: va creando los nuevos pods, comprueba que funcionan y se eliminan los antiguos.

~~~
paloma@coatlicue:~/kubernete$ kubectl create -f deployment.yaml 
deployment.apps/nginx created
paloma@coatlicue:~/kubernete$ kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE    IP           NODE       NOMINATED NODE   READINESS GATES
nginx-79f5849f8d-2vngt   1/1     Running   0          114s   172.17.0.4   minikube   <none>           <none>
nginx-79f5849f8d-7xm42   1/1     Running   0          114s   172.17.0.5   minikube   <none>           <none>
~~~

Actualización de .yaml:
~~~
paloma@coatlicue:~/kubernete$ kubectl apply -f deployment.yaml 
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
deployment.apps/nginx configured
paloma@coatlicue:~/kubernete$ kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
nginx-79f5849f8d-2vngt   1/1     Running   0          16m   172.17.0.4   minikube   <none>           <none>
nginx-79f5849f8d-7xm42   1/1     Running   0          16m   172.17.0.5   minikube   <none>           <none>
nginx-79f5849f8d-fbn8t   1/1     Running   0          23s   172.17.0.8   minikube   <none>           <none>
nginx-79f5849f8d-mbq4p   1/1     Running   0          23s   172.17.0.7   minikube   <none>           <none>
nginx-79f5849f8d-wl2qc   1/1     Running   0          23s   172.17.0.6   minikube   <none>           <none>
~~~



