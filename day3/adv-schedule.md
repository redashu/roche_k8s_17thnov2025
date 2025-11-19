# 🧱 1. Node Selector Example

Schedules a pod only on nodes with a given label.

## Step 1 — Label your node
```bash
kubectl label nodes <node-name> env=dev
```

## Step 2 — Pod using nodeSelector
```yaml
apiVersion: v1
kind: Pod
metadata:
    name: pod-nodeselector
spec:
    nodeSelector:
        env: dev
    containers:
        - name: app
            image: nginx
```

---

# 🧲 2. Node Affinity Example (Preferred + Required)

Node affinity is more flexible and recommended over nodeSelector.

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: pod-node-affinity
spec:
    affinity:
        nodeAffinity:
            # REQUIRED scheduling rule
            requiredDuringSchedulingIgnoredDuringExecution:
                nodeSelectorTerms:
                    - matchExpressions:
                            - key: nodegroup
                                operator: In
                                values:
                                    - backend

            # PREFERRED scheduling rule
            preferredDuringSchedulingIgnoredDuringExecution:
                - weight: 50
                    preference:
                        matchExpressions:
                            - key: zone
                                operator: In
                                values:
                                    - us-east-1a
    containers:
        - name: app
            image: nginx
```

**Example node labels:**
```bash
kubectl label node <node> nodegroup=backend zone=us-east-1a
```

---

# 🚫 3. Pod Anti-Affinity Example

Keep two pods away from each other (useful for HA apps).

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: pod-anti-affinity
    labels:
        app: web
spec:
    affinity:
        podAntiAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
                - labelSelector:
                        matchLabels:
                            app: web
                    topologyKey: kubernetes.io/hostname
    containers:
        - name: app
            image: nginx
```

This forces replicas of the same app onto different nodes.

---

# ☢️ 4. Taints & Tolerations Example

## Step 1 — Taint a node
```bash
kubectl taint nodes <node-name> dedicated=db:NoSchedule
```

## Step 2 — Pod with toleration
```yaml
apiVersion: v1
kind: Pod
metadata:
    name: pod-toleration
spec:
    tolerations:
        - key: "dedicated"
            value: "db"
            operator: "Equal"
            effect: "NoSchedule"

    containers:
        - name: app
            image: mysql:5.7
```

This pod can run on the tainted node, while other pods cannot.

---

# ⚡ 5. Resource Requests, Limits & PriorityClass

## Create a PriorityClass:
```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
    name: high-priority
value: 100000
globalDefault: false
description: "Demo high priority class"
```

## Pod with resource req/limits + priority:
```yaml
apiVersion: v1
kind: Pod
metadata:
    name: pod-resource-priority
spec:
    priorityClassName: high-priority
    containers:
        - name: app
            image: nginx
            resources:
                requests:
                    cpu: "250m"
                    memory: "256Mi"
                limits:
                    cpu: "500m"
                    memory: "512Mi"
```

---

# 🧪 6. Hands-On: Schedule Pod Using Node Affinity (DEMO)

This is the full, practical example you can run now.

## Step 1 — Label a node
```bash
kubectl label nodes <node-name> role=mysql
```

## Step 2 — Apply this pod
```yaml
apiVersion: v1
kind: Pod
metadata:
    name: mysql-affinity-demo
spec:
    affinity:
        nodeAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
                nodeSelectorTerms:
                    - matchExpressions:
                            - key: role
                                operator: In
                                values:
                                    - mysql
    containers:
        - name: mysql
            image: mysql:5.7
            env:
                - name: MYSQL_ROOT_PASSWORD
                    value: demo123
```

## Result

- Pod will **ONLY** run on the node labeled `role=mysql`
- If that node is unschedulable → pod stays in `Pending`
- No other node will host this pod