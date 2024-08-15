
## 1. Create Project

- Create nodejs project. In this example we are taking name of project as cars
- Configure service port, here we are taking port = `6001`
- Database url will be `mongodb://admin:password@mongo-service:27017/cars` where dbname = `cars`, usename = `admin`, password = `password`, host = `mongo-service`, port = `27017`.
- Create Dockerfile in root directory of the project

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

- Create docker image using command in the base directory of the project
  
  ```console
  docker build -t cars .
  ```
- Check newly created docker images using command
  
   ```console
  docker images
  ```

<br><br>

## 3. Deploy on minikube kubernetes

- Load cars docker image to minikube using command

  ```console
  minikube image load cars
  ```
- Check newly loaded minikube images using command
  
  ```console
  minikube image ls
  ```
   You can see `docker.io/library/cars:latest` in the list.
- We need a service deployment yaml file. Either we can manually create one yaml file or we can use below command to generate yaml file name `cars.yaml`

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
      app: hello-world-php
    name: hello-world-php
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: hello-world-php
    strategy: {}
    template:
      metadata:
        creationTimestamp: null
        labels:
          app: hello-world-php
      spec:
        containers:
        - image: hello-world-php
          name: hello-world-php
          imagePullPolicy: Never  # this is addition
          resources: {}
  status: {}
  ```
  
  
  

