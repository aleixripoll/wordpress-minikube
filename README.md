# Introduction

Wordpress setup in a minikube Kubernetes cluster using the folowing components:

- nginx reverse proxy (2 replicas)

- Wordpress (2 replicas)

- MySQL database

- Redis cache


# Setup 

$ kubectl create secret generic mysql-secrets --from-literal=rootpw=secretpw
secret "mysql-secrets" created

$ kubectl get secrets
NAME                  TYPE                                  DATA      AGE
mysql-secrets         Opaque                                1         12s

$ kubectl create -f mysql_vol.yaml
persistentvolumeclaim "mysql-volume-claim" created

$ kubectl create -f mysql.yaml
deployment.apps "mysql-deployment" created
service "mysql-service" created

$ kubectl get po
NAME                                READY     STATUS    RESTARTS   AGE
mysql-deployment-68ffdd65d5-fxw6z   1/1       Running   0          19s

$ kubectl create -f redis_vol.yaml 
persistentvolumeclaim "redis-volume-claim" created

$ kubectl create -f nfs_server.yaml 
deployment.apps "nfs-deployment" created
service "nfs-server" created

$ kubectl create -f nfs_pv.yaml 
persistentvolume "nfs-volume" created

$ kubectl get pv
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
kubernetes      ClusterIP   10.39.240.1     <none>        443/TCP                      24d
mysql-service   ClusterIP   10.39.254.121   <none>        3306/TCP                     1h
nfs-server      ClusterIP   10.39.247.226   <none>        2049/TCP,20048/TCP,111/TCP   40s
nfs-service     ClusterIP   10.39.248.83    <none>        2049/TCP,20048/TCP,111/TCP   5s
redis-service   ClusterIP   10.39.252.46    <none>        6379/TCP                     59m

$ kubectl create -f nfs_pvc.yaml 
persistentvolumeclaim "nfs-volume-claim" created

$ kubectl get pvc
NAME                      STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mysql-volume-claim        Bound     pvc-d0109d4e-bfbf-11e8-9dd2-42010a84009b   2Gi        RWO            standard       1h
nfs-server-volume-claim   Bound     pvc-2552fce1-bfc7-11e8-9dd2-42010a84009b   1Gi        RWO            standard       15m
nfs-volume-claim          Bound     nfs-volume                                 1Gi        RWX                           3s
redis-volume-claim        Bound     pvc-2b36f131-bfc0-11e8-9dd2-42010a84009b   1Gi        RWO            standard       1h

$ kubectl create -f wordpress.yaml 
deployment.apps "wordpress-deployment" created
service "wordpress-service" created

$ kubectl get po
NAME                                    READY     STATUS    RESTARTS   AGE
mysql-deployment-68ffdd65d5-fxw6z       1/1       Running   0          1h
nfs-deployment-94cf5c845-phgf5          1/1       Running   0          6m
redis-deployment-766f844b6c-flsq2       1/1       Running   0          1h
wordpress-deployment-5bfd754575-ft6kz   1/1       Running   0          14s
wordpress-deployment-5bfd754575-l8snv   1/1       Running   0          14s

$ kubectl create -f nginx.yaml 
configmap "nginx-configmap" created
deployment.apps "nginx-deployment" created
service "nginx-service" created

$ kubectl get services
NAME                TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)                      AGE
kubernetes          ClusterIP      10.39.240.1     <none>          443/TCP                      24d
mysql-service       ClusterIP      10.39.254.121   <none>          3306/TCP                     1h
nfs-service         ClusterIP      10.39.248.83    <none>          2049/TCP,20048/TCP,111/TCP   6m
nginx-service       LoadBalancer   10.39.252.217   35.205.31.129   80:32285/TCP                 49s
redis-service       ClusterIP      10.39.252.46    <none>          6379/TCP                     1h
wordpress-service   ClusterIP      10.39.255.130   <none>          80/TCP                       1m

$ curl -L -i 35.205.31.129
HTTP/1.1 302 Found
Server: nginx/1.15.3
Date: Mon, 24 Sep 2018 07:16:22 GMT
Content-Type: text/html; charset=UTF-8
Content-Length: 0
Connection: keep-alive
X-Powered-By: PHP/7.2.10
Expires: Wed, 11 Jan 1984 05:00:00 GMT
Cache-Control: no-cache, must-revalidate, max-age=0
Location: http://35.205.31.129/wp-admin/install.php

HTTP/1.1 200 OK
Server: nginx/1.15.3
Date: Mon, 24 Sep 2018 07:16:22 GMT
Content-Type: text/html; charset=utf-8
Transfer-Encoding: chunked
Connection: keep-alive
X-Powered-By: PHP/7.2.10
Expires: Wed, 11 Jan 1984 05:00:00 GMT
Cache-Control: no-cache, must-revalidate, max-age=0
Vary: Accept-Encoding

<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" lang="en-US" xml:lang="en-US">
<head>
	<meta name="viewport" content="width=device-width" />
	<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
	<meta name="robots" content="noindex,nofollow" />
	<title>WordPress &rsaquo; Installation</title>
    ...



