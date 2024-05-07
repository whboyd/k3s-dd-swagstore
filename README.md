# K3s Datadog Swagstore

This project contains  instructions for building a local K3s cluster using [k3d](https://k3d.io/v5.6.3/) and deploying the [Swagstore](https://github.com/smazzone/microservices-demo-multiarch) app to it.

## K3s Cluster Setup

***Note:*** You need Docker installed and a running Docker daemon to use k3d.

Download and execute the k3d install script.

```
curl -s https://raw.githubusercontent.com/rancher/k3d/main/install.sh | bash
```

Verify that k3d is installed and that you can run it.

```
$ k3d version
k3d version v5.6.3
k3s version v1.28.8-k3s1 (default)
```

Create a cluster called `localha` with a high-availability configuration of 3 server nodes and 1 agent node.

```
k3d cluster create localha --servers 3 --agents 1
```

Use kubectl to list the nodes in your cluster.

```
$ kubectl get nodes
NAME                   STATUS   ROLES                       AGE     VERSION
k3d-localha-agent-0    Ready    <none>                      2m23s   v1.28.8+k3s1
k3d-localha-server-0   Ready    control-plane,etcd,master   2m57s   v1.28.8+k3s1
k3d-localha-server-1   Ready    control-plane,etcd,master   2m41s   v1.28.8+k3s1
k3d-localha-server-2   Ready    control-plane,etcd,master   2m26s   v1.28.8+k3s1
```

## Deploy the Swagstore

Download the Swagstore K8s manifest.

```
curl -L https://raw.githubusercontent.com/smazzone/microservices-demo-multiarch/main/release/kubernetes-manifests.yaml > swagstore-manifests.yaml
```

Feel free to look over the manifests file first, then deploy it to the cluster.

```
kubectl apply -f swagstore-manifests.yaml
```

Check the status of the application's pods. Note that it may take a few moments for all of them to become ready. The application may not function properly until all of the components are ready.

```
$ kubectl get pods
NAME                                     READY   STATUS    RESTARTS   AGE
adservice-5b6957b769-8xxtm               1/1     Running   0          107s
cartservice-75875944d8-p65kc             1/1     Running   0          108s
checkoutservice-84dfb78cf-x8nf7          1/1     Running   0          108s
currencyservice-77c94d7dd7-j98wz         1/1     Running   0          107s
emailservice-5b96ffd557-szjw9            1/1     Running   0          108s
frontend-687886bfb7-72tzg                1/1     Running   0          108s
loadgenerator-67c8547b86-4nxl4           1/1     Running   0          108s
paymentservice-d5bb5c977-dgrn8           1/1     Running   0          108s
productcatalogservice-6bdb85f7f9-54q8j   1/1     Running   0          108s
recommendationservice-d58b64449-9p6z5    1/1     Running   0          108s
redis-cart-57d97f7b6f-sddrj              1/1     Running   0          107s
shippingservice-bd8cddd4f-fbl8h          1/1     Running   0          107s
```

Temporarily forward a port from the host machine to the frontend so that you can test the app in your browser.

***Note:*** You will need to leave this command running while you access the app.

```
kubectl port-forward deployment/frontend 8080:8080
```

Navigate to the application frontend in a browser.

```
localhost:8080
```

Congratulations! You have just stood up a highly-available K3s cluster and deployed a multi-component microservice application to it!

## Cleanup

If you want to keep your cluster but delete the Swagstore application, simply delete the Kubernetes objects using the manifest file:

```
kubectl delete -f swagstore-manifests.yaml
```

Otherwise, you can delete the entire K3s cluster using k3d.

```
k3d cluster delete localha
```

If you want to delete k3d, you only need to delete the binary.

```
sudo rm $(which k3d)
```