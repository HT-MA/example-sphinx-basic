KEDA
===================================



Overview
----------------

Install KEDA
---------------

Deploying KEDA with Helm.

  .. code-block:: shell
    
    helm repo add kedacore https://kedacore.github.io/charts
    helm repo update
    helm install keda kedacore/keda --namespace keda --create-namespace


Create deployment
------------------

  .. code-block:: yaml

    apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: "0.02" # ??CPU???0.5?
            memory: "5Mi" # ???????256Mi
          requests:
            cpu: "0.01" # ??CPU???0.2?
            memory: "2Mi" # ???????128Mi

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: NodePort
  

Create ScaledObject
-------------------

  .. code-block:: yaml

    apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: nginx-mem-scaler
spec:
  advanced:
    horizontalPodAutoscalerConfig:
      behavior:
        scaleDown:
          policies:
          - type: Pods
            value: 1
            periodSeconds: 10
          stabilizationWindowSeconds: 0
  scaleTargetRef:
    name: nginx-deployment
  triggers:
    - type: cron
      metadata:
        timezone: Asia/Shanghai
        start: 55 13 * * *       
        end: 20 14 * * *         
        desiredReplicas: "6"
    - type: cron
      metadata:
        timezone: Asia/Shanghai 
        start: 30 14 * * *       
        end: 45 14 * * *         
        desiredReplicas: "3"
    - type: cpu
      metadata:
      # Required
       type: Utilization # Allowed types are 'Utilization' or 'AverageValue'
       value: "10"
  minReplicaCount: 1  # ?????
  maxReplicaCount: 10  # ?????


  
