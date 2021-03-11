# Production

Create a Kubernetes secret resource for those service account credentials. It might look something like this:

```
apiVersion: v1
kind: Secret
metadata:
  name: my-data-service-account-credentials
type: Opaque
data:
  sa_json: <contents of running 'base64 the-downloaded-SA-credentials.json'>
```

Convert the Key to base64

```
base64 <keyname>
```

Mount the credentials in the container that needs access:

```
[...]
spec:
  containers:
  - name: my-container
    volumeMounts:
    - name: service-account-credentials-volume
      mountPath: /etc/gcp
      readOnly: true
[...]
  volumes:
  - name: service-account-credentials-volume
    secret:
      secretName: my-data-service-account-credentials
      items:
      - key: sa_json
        path: sa_credentials.json

```

Set the GOOGLE_APPLICATION_CREDENTIALS environment variable in the container to point to the path of the mounted credentials:

```
[...]
spec:
  containers:
  - name: my-container
    env:
    - name: GOOGLE_APPLICATION_CREDENTIALS
      value: /etc/gcp/sa_credentials.json
```
