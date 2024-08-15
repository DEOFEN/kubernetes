
## 1. Create Project

1. Create nodejs project. In this example we are taking name of project as cars
<br>

3. Configure service port, here we are taking port = `6001`.
<br>

5. Database url will be `mongodb://admin:password@mongo-service:27017/cars` where dbname = `cars`, usename = `admin`, password = `password`, host = `mongo-service`, port = `27017`.
<br>

6. Create Dockerfile in root directory of the project

  ```Dockerfile
  FROM public.ecr.aws/docker/library/node:22-alpine

  WORKDIR /usr/src/app

  COPY . .

  RUN npm install

  RUN npm run build

  EXPOSE 6001

  CMD ["node" , "dist/main"]
  ```

<br><br>

## 2. Create docker image

1. Create docker image using command in the base directory of the project
  
  ```console
  docker build -t cars .
  ```
<br>

2. Check newly created docker images using command
  
  ```console
  docker images
  ```

<br><br>

## 3. Deploy on minikube kubernetes

1. Load cars docker image to minikube using command

  ```console
  minikube image load cars
  ```
<br>

2. Check newly loaded minikube images using command
  
  ```console
  minikube image ls
  ```
   You can see `docker.io/library/cars:latest` in the list.
<br><br>

3. We need a service deployment yaml file. Either we can manually create one yaml file or we can use below command to generate yaml file name `cars.yaml`

  ```console
  kubectl create deployment cars --image=cars --dry-run=client -o yaml > cars.yaml
  ```
  yaml file will look like this, add `imagePullPolicy: Never` to avoid image pull on service restart
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    creationTimestamp: null
    labels:
      app: cars
    name: cars
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: cars
    strategy: {}
    template:
      metadata:
        creationTimestamp: null
        labels:
          app: cars
      spec:
        containers:
        - image: cars
          name: cars
          imagePullPolicy: Never  # update required
          resources: {}
  status: {}
  ```
<br>

4. Create kubernetes deployment using
  
  ```console
  kubectl apply -f cars.yaml
  ```
<br>

5. Verify kubernetes deployments using
  
  ```console
  kubectl get deployments
  ```
<br>

6. Verify pods using
  
  ```console
  kubectl get pods
  ```
<br>

7. Create service or Expose application to host machine using
  
  ```console
  kubectl expose deployment/imfi --type="NodePort" --port 6001
  ```
<br>

8. Verify service 
  
  ```console
  kubectl get services
  ```
<br>

9. Get service url using command
  
  ```console
  minikube service cars --url
  ```
Example output: `http://127.0.0.1:50565`
<br><br>

10. Test application using
  
  ```console
  curl http://127.0.0.1:50565
  ```
<br>


  
  
  
  

