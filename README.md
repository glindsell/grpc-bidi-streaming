# Routeguide-app
## Description  
This app demonstrates a GRPC server and client communicating using bidirectional streaming in a single node k8s cluster, with load balancing attempted between three pods using Linkerd - LOAD BALANCING FAILS.

## Run the app
Follow the instructions to install minikube:  
https://kubernetes.io/docs/tasks/tools/install-minikube/

Start minikube:  
```minikube start```

Set docker env:  
```eval $(minikube docker-env)```

Build docker image:  
```docker build -t routeguide-app .```

Run local image as client in minikube:  
```kubectl run routeguide-app-client --image=routeguide-app:latest --port=10000 --image-pull-policy=Never```

Check that it's running:  
```kubectl get pods```

Install Linkerd:  
```curl -sL https://run.linkerd.io/install | sh```

Add linkerd to your path:  
```export PATH=$PATH:$HOME/.linkerd2/bin```

Verify the CLI is installed and running correctly:  
```linkerd version```

To check that your cluster is configured correctly and ready to install the control plane, you can run:  
```linkerd check --pre```

Install the lightweight control plane into its own namespace (linkerd):  
```linkerd install | kubectl apply -f -```

Validate that everything’s happening correctly:  
(This command will patiently wait until Linkerd has been installed and is running.)  
```linkerd check```

In a new terminal view the Linkerd dashboard by running:  
```linkerd dashboard```

Back in terminal view deployments:  
```kubectl -n linkerd get deploy```

Install the app as a new deployment (using docker hub image at glindsell/helloworld-app):  
```linkerd inject routeguide-app-grpc.yml | kubectl apply -f -```

View new deployment in the Linkerd dashboard.  

Expose the deployment as a service:  
```kubectl expose deployment routeguide-app-server --type=NodePort```

Get IP Address of services:  
```kubectl get services```

Exec into client:  
```kubectl exec -it routeguide-app-client-<id-goes-here> -- /bin/bash```

cd into app dir:  
```cd src/routeguide-app/```

Change <IP-ADDR> to CLUSTER-IP of helloworld-app-server (line 32 of helloworld-app/greeter_client):  
```vim client/client.go```  
change:  
```address = "<IP-ADDR>:10000"```

Run the client to send requests:  
```go run client/client.go```

Observe that no load balancing occurs over server pods.  

Click on Grafana icon to view individual pod’s stats, and view pod logs to confirm.  

To bring everything down:  
```minikube stop```  
```minikube delete```

Note...
TLS is available via:  
```
go run server/server.go -tls=true
```

and

```
go run client/client.go -tls=true
```
