# [@matwolbec tutorials](https://matwolbec.github.io/tutorials/)

Back to [page 7](terraform-azure-7.md)

## Custom application
We will create a simple docker image to customize the HTML and see it running

## Requirements
- Docker [https://docs.docker.com/engine/install/](https://docs.docker.com/engine/install/)
- Docker Hub Account [https://hub.docker.com/](https://hub.docker.com/)

## Dockerfile
Let's create a Dockerfile and push it to Dockerhub.
```bash
mkdir docker
cd docker
touch Dockerfile
```

Open the file and add:
```docker
FROM php:7.4-apache
COPY . /var/www/html/
```

Create a ```index.php``` file. We want to PHP print the hostname, so we will know what host we are getting to:
```bash
touch index.php
```

Add the content:
```php
<?php
echo gethostname();
?>
```

## Building the docker image
Now you should have your dockerhub account. Let's build. Don't forget to substitute the ```<your-dockerhub-name>``` by your account. For example, mine is ```matwolbec```.

```bash
docker build -t <your-dockerhub-name>/php-gethostname:v1 -f Dockerfile .
```

Before we can push it, let's create the repository on DockerHub: [https://hub.docker.com/repository/create](https://hub.docker.com/repository/create).
Name it ```php-gethostname``` and leave the visibility as ```Public```.

Login on your docker-cli:
```bash
docker login
```

We can now push it to dockerhub:
```bash
docker push matwolbec/php-gethostname:v1
```

We should tag the version as the latest available:
```bash
docker tag matwolbec/php-gethostname:v1  matwolbec/php-gethostname:latest
docker push matwolbec/php-gethostname:latest
cd ..
```

## Deploying our image to our Kubeernetes cluster
Open the ```deployment.yaml``` and change:
```
          image: httpd:latest
```

To:

```
          image: matwolbec/php-gethostname:latest
```

And apply:
```bash
kubectl apply -f deployment.yaml
```

You can watch the magic happening with:
```bash
kubectl get pods
```

## Next steps

Go to [Ingress Controller](terraform-azure-9.md).
