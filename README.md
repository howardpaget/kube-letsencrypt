# kube-nginx-letsencrypt

Obtain and install Let's Encrypt certificates for Kubernetes Ingresses

https://hub.docker.com/r/sjenning/kube-nginx-letsencrypt/

## Example Kubernetes Config

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: your-domain-letsencrypt-job
  labels:
    app: your-domain-letsencrypt
spec:
  template:
    metadata:
      name: your-domain-letsencrypt
      labels:
        app: your-domain-letsencrypt
    spec:
      containers:
        - image: ghcr.io/howardpaget/dotly-artefacts/arm/kube-letsencrypt:latest
          name: your-domain-letsencrypt
          imagePullPolicy: Always
          ports:
            - containerPort: 80
          env:
            - name: DOMAIN
              value: "*.your-domain.com"
            - name: EMAIL
              value: email@your-domain.com
            - name: SECRET
              value: your-domain-letsencrypt-certs
            - name: DEPLOYMENT
              value: your-domain-api
#            - name: DRY_RUN
#              value: dryrun
      restartPolicy: Never
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: your-domain-ingress
spec:
  tls:
    - hosts:
        - "*.your-domain.com"
      secretName: letsencrypt-certs
  rules:
    - host: www.your-domain.com
      http:
        paths:
          - path: /.well-known/
            pathType: Prefix
            backend:
              service:
                name: your-domain-letsencrypt
                port:
                  number: 80
```