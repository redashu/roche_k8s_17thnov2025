# Splunk Kubernetes Deployment Tasks

1. Create a YAML file named `<yourname>splunk.yaml`
2. When you **RUN** the YAML, it must create a namespace called `<yourname>splunkns` and all resources must be inside this namespace only
3. Use `splunk/splunk:latest` image from Docker Hub to create deployment
4. Required ENV variables must be stored in **ConfigMap**
5. Required password must be stored in **Secret**
6. Replica count: `1`
7. Create service of **NodePort** type named `<yourname>svc`
8. Access this from web browser
9. **Note:** Default username of Splunk is `admin` and Splunk default port number is `8000`