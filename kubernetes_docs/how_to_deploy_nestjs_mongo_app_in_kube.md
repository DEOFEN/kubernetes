
## 1. Deploy mongodb service in kubernetes cluster

1. Create mongodb deployment file
   
   ```yaml
   # mongo.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: mongo
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: mongo
     template:
       metadata:
         labels:
           app: mongo
       spec:
         containers:
         - name: mongo
           image: mongo:5.0
           ports:
           - containerPort: 27017
           env:
           - name: MONGO_INITDB_ROOT_USERNAME
             value: "admin"
           - name: MONGO_INITDB_ROOT_PASSWORD
             value: "password"
   ```
<br>

2. Create mongodb service file
   
   ```yaml
   # mongo-service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: mongo-service
   spec:
     selector:
       app: mongo
     ports:
     - protocol: TCP
       port: 27017
       targetPort: 27017
     type: ClusterIP
   ```
<br>

3. Apply configurations for above files using command 
   
   ```console
   kubectl apply -f mongo.yaml
   kubectl apply -f mongo-service.yaml
   ```
<br>

4. Verify service 
  
     ```console
     kubectl get services
     ```
<br>

5. For persistant storage update deployments file and create svc file
   
   ```yaml
   # mongo.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: mongo
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: mongo
     template:
       metadata:
         labels:
           app: mongo
       spec:
         containers:
         - name: mongo
           image: mongo:5.0
           ports:
           - containerPort: 27017
           env:
           - name: MONGO_INITDB_ROOT_USERNAME
             value: "admin"
           - name: MONGO_INITDB_ROOT_PASSWORD
             value: "password"
   ```

   ```yaml
   # mongo-pvc.yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: mongo-pvc
   spec:
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 1Gi
   ```
<br>

6. Apply svc configurations 
   
   ```console
   kubectl apply -f mongo-pvc.yaml
   kubectl apply -f mongo.yaml
   ```

<br><br>


## 2. Create Project

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

## 3. Create docker image

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

## 4. Deploy on minikube kubernetes

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


  
  
  
  

