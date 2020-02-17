Instalación:
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

