
# LAB for creating a secret from command line.
   
## Encrypt the variables
```
echo -n database | base64
```
``` 
echo -n Mysql123 | base64
```

### For your references. 
```
[root@master1 volume]# echo -n database | base64 
ZGF0YWJhc2U=
[root@master1 volume]# echo -n Mysql123 | base64 
TXlzcWwxMjM=
[root@master1 volume]# 
```

## How to create Namespace from Yaml file?
Step 1: 
```yaml 
cat <<EOF>>tiger.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: tiger
EOF
```
Step 2:
```
kubectl create -f tiger.yaml
```
## Create a secret
### Step 1: Create a secret under "tiger" namespace by command line.

```
kubectl -n tiger create secret generic prod-db-secret4 --from-literal=username='database' --from-literal=password='Mysql123'
```

## How to get more details of this secret?
### Step 1: With the help of Discribe coommand.
```
kubectl describe secrets -n tiger prod-db-secret4
```
### Step 2: With the help of "get" sub command.
```
kubectl -n tiger get secrets prod-db-secret4
```
## 1. POD consuming secret by volume.

### Step 1: Creating a POD under namespace **tiger** and insert the variables from **volume**.
```
cat <<EOF>>secret-volume-pod4.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-volume-pod4
  namespace: tiger
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: prod-db-secret4
      optional: true
EOF
```
### Step 2: Create the POD
```
kubectl apply -f secret-volume-pod4.yaml
```
```
kubectl get pods -n tiger
```
### Chekc the POD

#### Login into the POD and check the files are created under "/etc/foo" directory.
```
kubectl -n tiger exec -it $(kubectl get pods -n tiger | grep secret-volume-pod4) -- ls -ltr /etc/foo/ ; echo
```
#### use the command "cat" to list the content of username file.
```
kubectl -n tiger exec -it $(kubectl get pods -n tiger | grep secret-volume-pod4) -- cat /etc/foo/username ; echo
```
#### use the command "cat" to list the content of password file.
```
kubectl -n tiger exec -it $(kubectl get pods -n tiger | grep secret-volume-pod4) -- cat /etc/foo/password ; echo
```


# Question:

You are tasked to create a secret and consume the secret in a pod using envireonment variables as follows
- Create a secret name app-secret1 with a key/value pair.  key30/value4
- Start an Nginx POD named nginx-secret1 using container image Nginx, and add an environment a variable exposing the value of the secret key key30 using BEST_VARIABLE1 as the name of the envireonment variable inside the pod. 

Solution:

google: https://kubernetes.io/
click on documentation
search secret
```
kubectl create secret generic app-secret1 --from-literal=key30=value4
```

```yaml
cat <<EOF>>q2-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-secret1
spec:
  containers:
  - name: mycontainer
    image: nginx
    env:
      - name: BEST_VARIABLE1
        valueFrom:
          secretKeyRef:
            name: app-secret1
            key: key30
  restartPolicy: Never
  EOF
```

```
kubectl create -f q2-pod.yaml
```

```
kubectl get pods
```

## Check the variable
```
kubectl exec -it nginx-secret1 -- env | grep BEST_VARIABLE1
```


## Clear the Lab
```
kubectl delete -n tiger pods/secret-volume-pod4 secrets/prod-db-secret4 --force --grace-period=0
kubectl delete namespaces tiger --force --grace-period=0
rm -f secret-volume-pod4.yaml  tiger.yaml
ls -ltr
```

## Delete the question lab.
```
kubectl delete -f q2-pod.yaml
kubectl delete secrets app-secret1
rm -f q2-pod.yaml
ls -ltr
```

   
