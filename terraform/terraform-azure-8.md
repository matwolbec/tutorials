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
FROM php:7.4-cli
COPY . /usr/src/myapp
WORKDIR /usr/src/myapp
CMD [ "php", "./index.php" ]
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

## Next steps

Go to [Ingress Controller](terraform-azure-9.md).
