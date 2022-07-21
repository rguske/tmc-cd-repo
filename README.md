# tmc-cd-repo

This repository includes Kubernetes kustomization files which are used to demo the new Tanzu Mission Control Continues Delivery feature.

**Notes:**

## Create a deployment.yaml file

cat <<EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blog
  namespace: ghost
  labels:
    app: blog
spec:
  replicas: 1
  selector:
    matchLabels:
      app: blog
  template:
    metadata:
      labels:
        app: blog
    spec:
      containers:
      - name: blog
        image: ghost:3.25.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 2368
        env:
        - name: url
          value: http://my-blog.corp.local
        volumeMounts:
        - mountPath: /var/lib/ghost/content
          name: content
      volumes:
      - name: content
        persistentVolumeClaim:
          claimName: blog-content
EOF

## Create a service.yaml file

cat <<EOF > service.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    name: blog
  name: blog
  namespace: ghost
spec:
  ports:
    - port: 80
      targetPort: 2368
  selector:
    app: blog
  type: LoadBalancer
EOF

# Create a pvc.yaml file
cat <<EOF > pvc.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: blog-content
  namespace: ghost
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-path
  resources:
    requests:
      storage: 5Gi
EOF

## Create a kustomization.yaml composing them

cat <<EOF >./kustomization.yaml
resources:
- deployment.yaml
- service.yaml
- pvc.yaml
EOF