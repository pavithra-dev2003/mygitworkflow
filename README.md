# Helm_Chart
A Helm Chart is a package that contains all the Kubernetes YAML files needed to run an app.It also has a values.yaml file where you can change settings easily (like image, port, replicas) without touching the main YAMLs.
## WebApp Helm Chart
This Helm chart deploys a MySQL database and a User Management Web Application on Kubernetes. The chart uses Helm templates for Deployments, Services, PersistentVolumeClaims, and ConfigMaps to make the deployment dynamic and configurable.
## Chart Structure

```
webappchart/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── configure.yaml
    ├── mysql-deployment.yaml
    ├── mysql-service.yaml
    ├── mysql-pvc.yaml
    ├── usermgmt-webapp-deployment.yaml
    └── usermgmt-webapp-service.yaml
```
Chart Files Explanation
### Chart.yaml
Contains metadata and overveiw about the Helm chart:
```
apiVersion: v2
name: webappchart
description: A Helm chart for MySQL and User Management Web App
type: application
version: 0.1.0
appVersion: "1.1.0"
```
apiVersion: Helm v2 for Helm 3 charts.
name: Chart name.
description: Short description of what the chart does.
type: application indicates deployable app.
version: Helm chart version.
appVersion: Version of the application being deployed.

### values.yaml

Contains default values used in the Helm templates:
#### configure
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: usermanagement-dbcreation-script
data: 
  mysql_usermgmt.sql: |-
    DROP DATABASE IF EXISTS webappdb;
    CREATE DATABASE webappdb;
```
Creates a ConfigMap containing a SQL script.
Drops any existing webappdb database and creates a new one.
This script will be mounted into the MySQL container to initialize the database on startup.
#### MySQL Deployment
```
mysql:
  image: "mysql:5.6"
  replicas: 1
  containerPort: 3306
  pvcName: "my-pvc"
  rootPassword: "dbpassword11"
  storage: 3Gi
```
image: MySQL Docker image.
replicas: Number of MySQL pods.
containerPort: Port MySQL listens on inside the pod.
pvcName: Name of the PersistentVolumeClaim for storing database data.
rootPassword: Root password for MySQL.
storage: Amount of storage requested for the PVC.
Configures MySQL image, container port, PVC, root password, and storage.

#### User Management Web App
```
usermgmtWebApp:
  image: "stacksimplify/kube-usermgmt-webapp:1.0.0-MySQLDB"
  replicas: 3
  containerPort: 8080
  initContainerImage: "busybox:1.31"
  db:
    name: webappdb
    username: root
    password: dbpassword11
    hostname: mysql
    port: 3306
```
image: Docker image of the web application.
replicas: Number of web app pods.
containerPort: Port on which the web app container listens.
initContainerImage: Image for the init container that waits for MySQL readiness.
db: Database connection details (name, username, password, host, port).
Configures the app image, replicas, port, init container (checks MySQL readiness), and database credentials.

#### MySQL Service
```
mysqlservice:
  service:
    port: 80
    targetPort: 3306
    clusterIP: "None"
```
Creates a headless service for MySQL.
Allows the web app pods to discover and connect to MySQL via DNS.
clusterIP: "None" makes it headless (no load balancing).
Headless service to allow the web app to connect to MySQL.

#### User Management Web App Service
```
userwebappservice:
  type: LoadBalancer
  port: 80
  targetport: 8080
```
Exposes the web application externally via a LoadBalancer service.
port: External port to access the app.
targetport: Internal container port the service maps to.
Exposes the web app externally via LoadBalancer.

 ### temple with containes the files

  1)templates/configure.yaml
  Contains a ConfigMap that holds the SQL script (mysql_usermgmt.sql) to initialize the MySQL database.
  On MySQL container startup, this script automatically drops the existing webappdb database (if any) and creates a new one.
  Ensures the database is ready before the web application starts.
 
  2)emplates/mysql-deployment.yaml

Helm template to deploy MySQL:
Uses values from values.yaml.
Configures container image, root password, ports, replicas.
Mounts PersistentVolumeClaim (/var/lib/mysql) to store database data.
Mounts a ConfigMap containing SQL scripts to /docker-entrypoint-initdb.d for DB initialization.

3)templates/mysql-pvc.yaml

Helm template to create a PersistentVolumeClaim (PVC):
Ensures MySQL data persists even if the pod restarts.
Uses dynamic values from values.yaml (storage size, PVC name).
Access mode ReadWriteOnce is suitable for a single MySQL pod.

4)templates/mysql-service.yaml

Helm template for MySQL Service:
Headless service (clusterIP: None) for internal pod-to-pod communication.
Routes traffic to MySQL pods using the selector app: mysql.
Ports are configurable via values.yaml.

 5)templates/usermgmt-webapp-deployment.yaml

Helm template for User Management Web App Deployment:
Uses initContainers to wait until MySQL is ready before starting the app.
Passes DB credentials and connection info as environment variables.
Configurable image, replicas, and container port from values.yaml.

6)templates/usermgmt-webapp-service.yaml

Helm template for User Management Web App Service:
Exposes the web app externally (LoadBalancer) or internally (ClusterIP/NodePort).
Maps external service port to the container port of the app.
Selector app: usermgmt-webapp ensures traffic is routed to correct pods.

### Deployment Instructions
```
#install the helm taken from open 
helm install <applicationname> ./<chartfoler>
#list the heml
helm list
#View Release History: To see the available revisions for a release:
 helm history <helmappname>
#make changes or update changes
helm upgrade <chartname> ./<chartfoler>
#rollback
helm rollback <helmappname> <revisionno>
#debugg
helm template --debug <helmlocation>
```

#### Access the web app

If type: LoadBalancer, use kubectl get svc usermgmt-webapp-service to find the EXTERNAL-IP.
If type: NodePort, use <Node-IP>:<NodePort>.
