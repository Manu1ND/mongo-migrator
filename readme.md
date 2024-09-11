### 1. **Starting Minikube**
```bash
minikube start
```

### 2. **Installing Helm**
```bash
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

### 3. **Adding Bitnami Repository**
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

### 4. **Deploying PostgreSQL**
```bash
helm install my-postgresql bitnami/postgresql --set postgresqlPassword=mysecretpassword
```

### 5. **Deploying MongoDB**
```bash
helm install my-mongodb bitnami/mongodb --set mongodbRootPassword=mysecretpassword
```

### 6. **Creating Migrator Deployment YAML**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb-relational-migrator
spec:
  replicas: 1
  selector:
    matchLabels:
      app: migrator
  template:
    metadata:
      labels:
        app: migrator
    spec:
      containers:
      - name: migrator
        image: public.ecr.aws/v4d7k6c9/relational-migrator:latest
        ports:
        - containerPort: 8080
        env:
        - name: PGSQL_HOST
          value: "my-postgresql"
        - name: MONGODB_HOST
          value: "my-mongodb"
```

### 7. **Creating Migrator Service YAML**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongodb-relational-migrator
spec:
  type: NodePort
  ports:
  - port: 8080
    targetPort: 8080
    nodePort: 30000
  selector:
    app: migrator
```

### 8. **Applying YAML**
```bash
kubectl apply -f migrator-deployment.yaml
kubectl apply -f migrator-service.yaml
```

### 9. **Checking Services**
```bash
kubectl get services
```

### 10. **Checking Pods**
```bash
kubectl get pods
```

### 11. **Port Forwarding for Local Access**
```bash
kubectl port-forward svc/my-postgresql 5432:5432
kubectl port-forward svc/my-mongodb 27017:27017
kubectl port-forward svc/mongodb-relational-migrator 8080:8080
```

### 12. **Retrieving PostgreSQL Password**
```bash
kubectl get secret --namespace default my-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d
```

### 13. **Retrieving MongoDB Root Password**
```bash
kubectl get secret --namespace default my-mongodb -o jsonpath="{.data.mongodb-root-password}" | base64 -d
```

### 14. **Insert Sample Data in PGSQL**
```bash
kubectl exec -it <postgresql-pod-name> -- psql -U postgres

CREATE DATABASE sample_mongo_mig;
\c sample_mongo_mig
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100),
  email VARCHAR(100) UNIQUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO users (name, email) VALUES
('Naik', 'naik@example.com'),
('Ganjoo', 'ganjoo@example.com');
SELECT * FROM users;
\q
```

### 15. **Open MongoDB Relational Migrator**
```
http://localhost:8080/
```

### 16. **Connection Details**
```plaintext
PostgreSQL: postgres
MongoDB: root

mongodb://new_user:new_password@my-mongodb:27017/sample_mongo_mig?authSource=admin
```