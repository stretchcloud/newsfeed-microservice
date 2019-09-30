# Newsfeed Microservice

This project contains three services:

* `quotes` which serves a random quote from `quotes/resources/quotes.json`
* `newsfeed` which aggregates several RSS feeds together
* `front-end` which calls the two previous services and displays the results.

## Prerequisites

* Docker
* Kubernetes Cluster



## Building

To build the Docker images of the different services, use the uploaded dockerfiles and build the container images.

###### dockerfile.frontend

```dockerfile
FROM openjdk:8
COPY . /usr/app/
WORKDIR /usr/app
EXPOSE 8080
CMD ["java", "-jar", "build/front-end.jar"]
```



###### dockerfile.newsfeed

```dockerfile
FROM openjdk:8
COPY . /usr/app/
WORKDIR /usr/app
EXPOSE 8080
CMD ["java", "-jar", "build/newsfeed.jar"]
```



###### dockerfile.quotes

```dockerfile
FROM openjdk:8
COPY . /usr/app/
WORKDIR /usr/app
EXPOSE 8080
CMD ["java", "-jar", "build/quotes.jar"]
```



Let's build the containers, tag it appropriately and push it to docker hub.

```bash
$ docker build -f Dockerfile.quotes -t newsfeed:1.0 .
$ docker build -f Dockerfile.newsfeed -t newsfeed:1.0 .
$ docker build -f Dockerfile.frontend -t frontend:1.0 .
$ docker tag quotes:1.0 jit2600/quotes:latest
$ docker tag newsfeed:1.0 jit2600/newsfeed:latest
$ docker tag frontend:1.0 jit2600/frontend:latest
$ docker push jit2600/quotes:latest
$ docker psuh jit2600/newsfeed:latest
$ docker push jit2600/newsfeed:latest
$ docker push jit2600/frontend:latest
```



## Running

All the apps take environment variables to configure them and expose the URL `/ping` which will just return a 200 response that you can use with e.g. a load balancer to check if the app is running.

### Front-end app

*Environment variables*:

* `APP_PORT`: The port on which to run the app
* `STATIC_URL`: The URL on which to find the static assets
* `QUOTE_SERVICE_URL`: The URL on which to find the quote service
* `NEWSFEED_SERVICE_URL`: The URL on which to find the newsfeed service
* `NEWSFEED_SERVICE_TOKEN`: The authentication token that allows the app to talk to the newsfeed service. This should be treated as an application secret. The value should be: `T1&eWbYXNWG1w1^YGKDPxAWJ@^et^&kX`

### Quote service

*Environment variables*

* `APP_PORT`: The port on which to run the app

### Newsfeed service

*Environment variables*

* `APP_PORT`: The port on which to run the app



### Deploy the Microservices

Deploy the newsfeed app on a K8S Cluster.

```bash
$ kubectl create -f quotes-deploy.yaml
$ kubectl create -f quotes-service.yaml 

$ kubectl create -f newsfeed-deploy.yaml
$ kubectl create -f newsfeed-service.yaml

$ kubectl create -f secret.yaml
$ kubectl create -f frontend-deploy.yaml
$ kubectl create -f frontend-service.yaml 
```



Let's access the service load balancer url to see whether the application is up and running.

```bash
$ export SERVICE_IP=$(kubectl get svc --namespace default frontend --template "{{ range (index .status.loadBalancer.ingress 0) }}{{ . }}{{ end }}")
$ echo http://$SERVICE_IP:8080
```



You should see a output similar to this

![Screen Shot 2019-09-30 at 16.04.38]([https://github.com/stretchcloud/newsfeed-microservice/blob/master/Screen%20Shot%202019-09-30%20at%2016.04.38.png](https://github.com/stretchcloud/newsfeed-microservice/blob/master/Screen Shot 2019-09-30 at 16.04.38.png))