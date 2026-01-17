### Monitor Our Own Nodejs Application using Prometheus stack

### For our own application, there is no exporter or ready application that we can just deploy on the cluster and scraping the metrics

#### We ourselves have to define the metrics

#### What is solution ?

#### Inorder to monitor our own application using prometheus, we need to prometheus client libraries in our application.

#### Prometheus client libraries are libraries for different programming languages, that give us abstract interface for defining metrics in our application that we want to expose as well as exposing those metrics in the time series format that prometheus can scrape.

#### Counter, Gauge, Historgram, Summary

#### How to expose metrics for our nodejs application using prometheus client library for nodejs, once we have exposed the metrics using the prometheus client library, we gonna deploy our simple nodejs application in the cluster and we gonna configure prometheus to start scraping the metrics that our application is exposing through this prometheus client library

### Basically we gonna configure prometheus to scrape a new target (ServiceMonitor), new target in this case will be the node.js application. Once we have prometheus scraping the metrics of our own application, we can configure alert rules, create a grafana dashboard based on those metrics to visualize the data.

#### Exposing Metrics - Nodejs Client Library
    
#### We are exposing 2 metrics that we are interested in.
    1. Number of requests, the application is getting to check anypoint in time how much load the application has
    2. Duration of Requests, basically how long does the appliation take to handle the request
    
#### In real project, we will ask developers to expose the metrics in the application, so that we can start monitoring it with prometheus, cuz developers know the code, they will basically go and find the prometheus client library, integrate with their code and expose some of the metrics that we are interested in to monitor. As a devops engineer, it not our responsibility, but developers team job.

#### We are creating two types of Metrics

#### httpRequestTotal
    Counter type -- A cumulative metric, whose value can only increase, resets to zero when the process restarts
    
#### httpsRequestDurationSeconds
    Historgram type -- It works in buckets, so we have multiple buckets, the lowest one and the highest one and depending on the value of Duration in seconds, it will assigned to one of the buckets
    
#### We want to track these metrics, basically whenever a request is made to application, we want to increment a no of requests or for that request, we want to calculate how long that request took.. So we need to define the metric and track the value in our logic

#### /metrics end point should be there in code logic or exposed for us to scrape

##### Metrics
The project exposes metrics on /metrics endpoint

To run the nodejs application:

    npm install 
    node app/server.js
    
#### We are building a docker image of the application and pushing it to private repository so that we can deploy it in kubernetes

To build the project:

    docker build -t dockerhub-repo-name/image-name:image-tag .
    docker login
    docker push dockerhub-repo-name/image-name:image-tag
    
#### 1. If you have CI/CD pipeline set up, just commit to the git repository, it will then trigger the build and the build will push it to the docker private repository, In this project, I just pushed it directly from my local computer to dockerhub for the sake of simplicity

#### 2. Deploy the application which is available in dockerhub (basically the image)
    a. create a kubernetes deployment and service configuration file for our nodejs app in the same repository as application (k8s-config.yaml)
    
#### 3. Because we are deploying a private repository image, we need to give kubernetes access to the docker private repository as well, so we need to create dockerlogin secret in the cluster that deployment can use when to trying to fetch the image from the docker hub repository
    kubectl create secret docker-registry my-registry-key --docker-server=https://index.docker.io/v1/ --docker-username=prashzan --docker-password=------
    
#### 4. We now have the secret (my-registry-key), configure that in the deployment so that kubernetes can fetch the image using the docker registry secret
    imagePullSecrets:
    - name: my-registry-key
```    
kubectl apply -f k8s-config.yaml
```    
#### 5. This will create nodejs app deployment and service in default namespace in cluster
    
#### 6. So to access the application on cluster using port forwarding
    kubectl get svc
    
    then apply
    
    kubectl port-forward svc/nodeapp 3000:3000
    
    Now accessible on localhost:3000 (this is basically sending the request to the application running inside the cluster
    
#### 7. How do we tell prometheus, that there is a new endpoint that it needs to scrape?
    Using ServiceMonitor Component

#### 8. ServiceMonitor is a link between prometheus and any new endpoint in our cluster that we want prometheus to monitor, that means we have to create a ServiceMonitor component for our nodejs application that points to that metrics endpoint, when we create this service monitor in the cluster, prometheus should automatically pick up a new target and start scraping those metrics from the new target.

```
kubectl apply -f k8-config.yaml
```
### 9. Run this command after creating deployment, service and service monitor component

### 10. Inside the kubernetes cluster, our nodejs-app and service monitor is running in default namespace and our prometheus stack is running in monitoring namespace

### 11. Lables are there in kubernetes config file, so that other resources can target that specific component (like deployment or service)
    
    
