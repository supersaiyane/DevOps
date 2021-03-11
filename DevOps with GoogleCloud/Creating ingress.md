
# This guide will show how to install nginx-ingress using helm

```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx
```

Once done check the service and IP
```
kubectl get svc ingress-nginx-controller

outPut

NAME                       TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)                      AGE
ingress-nginx-controller   LoadBalancer   10.92.26.121   34.94.165.154   80:31343/TCP,443:32755/TCP   11d
```

Now you can use this IP with domain names like 

*localtls:* A DNS server in Python3 to provide TLS to webservices on local addresses. It resolves addresses such as '192-168-0-1.yourdomain.net' to 192.168.0.1 and has a valid TLS certificate for them.
*xip.io / nip.io*
*sslip.io:* Alternative to this service, supports IPv6 and custom domains.

for more information click on the link below
[nip.io](https://nip.io/)

Usage in Yaml

```
kind: Ingress
metadata:
  name: ingress-resource
  namespace: rconn
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: 34.83.216.249.xip.io
    http:
      paths:
      - backend:
          serviceName: rconn
          servicePort: 5000
```





