# TLS/HTTPS

## TLS Secrets

Anytime we reference a TLS secret, we mean a PEM-encoded X.509, RSA (2048) secret.

You can generate a self-signed certificate and private key with:

```
$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ${KEY_FILE} -out ${CERT_FILE} -subj "/CN=${HOST}/O=${HOST}"
```

Then create the secret in the cluster via:
```
kubectl create secret rconntls ${CERT_NAME} --key ${KEY_FILE} --cert ${CERT_FILE}
```

The resulting secret will be of type kubernetes.io/tls

usage

you need to append tls array under spec in Ingress yaml

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
  tls:
     - hosts:
       - 34.83.216.249.xip.io
       secretName: rconntls
```

