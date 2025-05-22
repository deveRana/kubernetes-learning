Here's the improved guide with the corresponding YAML files included for each step:

---

# 🚀 Deploying Load-Balanced HTTP Echo Pods with a Shared Kubernetes Service

This guide walks you through deploying two distinct HTTP Echo pods (`rana` and `rushi`) using the `hashicorp/http-echo` image. Both pods will be load-balanced behind a single Kubernetes service called `unnati-service`. The pods are grouped under a common label (`class: unnati`), allowing the service to direct traffic to both.

---

## 🔧 Step 1: Create `rana` Deployment

Create a `rana` deployment using the `hashicorp/http-echo` image. This pod will respond with a message, "Hello from rana!"

```yaml
# rana-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rana
spec:
  replicas: 1
  selector:
    matchLabels:
      app: rana
      class: unnati
  template:
    metadata:
      labels:
        app: rana
        class: unnati
    spec:
      containers:
        - name: http-echo
          image: hashicorp/http-echo
          args:
            - "-text=Hello from rana!"
          ports:
            - containerPort: 5678
```

---

## 🔧 Step 2: Create `rushi` Deployment

Similarly, create a `rushi` deployment using the `hashicorp/http-echo` image. This pod will respond with a message, "Hello from rushi!"

```yaml
# rushi-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rushi
spec:
  replicas: 1
  selector:
    matchLabels:
      app: rushi
      class: unnati
  template:
    metadata:
      labels:
        app: rushi
        class: unnati
    spec:
      containers:
        - name: http-echo
          image: hashicorp/http-echo
          args:
            - "-text=Hello from rushi!"
          ports:
            - containerPort: 5678
```

---

## 📥 Step 3: Apply the Deployments

Now, apply both deployments to the Kubernetes cluster:

```bash
kubectl apply -f rana-deployment.yaml
kubectl apply -f rushi-deployment.yaml
```

This will create the `rana` and `rushi` pods in your cluster.

---

## 🌐 Step 4: Create the Shared Service

Create the Kubernetes service (`unnati-service`) that will load-balance traffic between the two pods. This service will expose both pods under a single IP.

```yaml
# unnati-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: unnati-service
spec:
  selector:
    class: unnati
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 5678
```

---

## 📥 Step 5: Apply the Service

Apply the service to your cluster:

```bash
kubectl apply -f unnati-service.yaml
```

This will create the `unnati-service`, which will route traffic to the `rana` and `rushi` pods.

---

## 🔍 Step 6: Verify the Setup

You can verify that everything is set up correctly by checking the following:

1. **Check the service details**:

   ```bash
   kubectl get svc unnati-service
   ```

2. **Check the pods and their labels**:

   ```bash
   kubectl get pods -l class=unnati -o wide
   ```

3. **Check the service endpoints**:

   ```bash
   kubectl get endpoints unnati-service
   ```

This will confirm that the service is correctly pointing to the two pods.

---

## ✅ Step 7: Test the Service

Finally, test the load-balanced service:

1. **Test by curling the service IP directly**:

   ```bash
   curl <service-ip>:8080
   ```

   Replace `<service-ip>` with the actual service IP obtained from the `kubectl get svc unnati-service` command. You will see responses like:

   ```
   Hello from rana!
   ```

   or

   ```
   Hello from rushi!
   ```

---

### ℹ️ Why Use Port 8080?

The service is configured to listen on port 8080, as defined in the `unnati-service.yaml` file. That’s why when you access the service, you need to specify `:8080` — this directs traffic to the service’s defined port, which then forwards to the pods’ target port (5678).

---
