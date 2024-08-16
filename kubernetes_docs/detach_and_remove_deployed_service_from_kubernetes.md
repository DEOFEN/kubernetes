# Detach_and_remove_deployed_service_from_kubernetes


## 1. Remove from kubernetes

1. Delete kubernetes service using command

     ```console
     kubectl delete service cars
     ```
<br>

2. Delete kubernetes deployment using command
  
     ```console
     kubectl delete deployment cars
     ```
<br>

3. Delete minikube image using command
  
     ```console
     minikube image rm cars
     ```
<br><br>

## 2. Remove from docker

1. Delete docker image

     ```console
     docker rmi cars:latest
     ```
<br>
