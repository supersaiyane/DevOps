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

# Development

This will be a always a temporary access as

- This is a temporary environment variable which is set in the gcloud shell of a perticular K8 cluster.
- Life of GOOGLE_APPLICATION_CREDENTIALS expires as you close down your gcloud shell.

```
export GOOGLE_APPLICATION_CREDENTIALS="/home/user/Downloads/my-key.json"
```

# In Docker

```
docker run \
  -e GOOGLE_APPLICATION_CREDENTIALS=/tmp/keys/[FILE_NAME].json \
  -v $(pwd):/tmp/keys/[FILE_NAME].json:ro \
  gcr.io/[PROJECT_ID]/[IMAGE]
```


