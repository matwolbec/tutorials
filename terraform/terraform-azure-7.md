# [@matwolbec tutorials](https://matwolbec.github.io/tutorials/)

Back to [page 5](terraform-azure-5.md)


##

apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-kubernetes
spec:
  selector:
    matchLabels:
      name: hello-kubernetes
  template:
    metadata:
      labels:
        name: hello-kubernetes
    spec:
      containers:
        - name: app
          image: paulbouwer/hello-kubernetes:1.8
          ports:
            - containerPort: 8080

## Next steps

Go to [deploy with jenkins](jenkins.md).


