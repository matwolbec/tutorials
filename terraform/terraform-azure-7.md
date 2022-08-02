# [@matwolbec tutorials](https://matwolbec.github.io/tutorials/)

Back to [page 6](terraform-azure-6.md)


## Creating a deployment file

The kubectl works with YAML definition. Create a file ```deployment.yaml```:
```bash
touch deployment.yaml
```

Open it and add:
```yaml
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
          image: httpd:latest
          ports:
            - containerPort: 80
```

Save and run:
```bash
kubectl apply -f deployment.yaml
```

You can check yours pods:
```bash
kubectl get pods
```

We didn'nt assign a public access yet, but you can run a proxy to connect and test. Get the name of the pod with the ```kubectl get pods``` and:
```bash
kubectl port-forward <name-of-your-pod> 8080:80
```

And access: [http://localhost:8080/](http://localhost:8080/)

It should show the message: It works!

## Adding a public access with Load Balancer

Add to your ```deployment.yaml```
```yaml
---

apiVersion: v1
kind: Service
metadata:
  name: hello-kubernetes
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 80
  selector:
    name: hello-kubernetes
    
```

And run:
```bash
kubectl apply -f deployment.yaml
```

You can check your services with:
```bash
kubectl get svc
```

Your public IP will be shown. Access it and Voilà!

## Destroying
Don't forget to destroy your resources to avoid charges.

Run:
```bash
kubectl delete -f deployment.yaml
terraform destroy
```



## Next steps

Go to [ingress controller](terraform-azure-8.md).


