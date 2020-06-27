# Creating a WebSphere Liberty OpenShift Application

In this lab, you'll create a simple application Deployment resource and deploy it to OpenShift.

To get started, log into OpenShift using the CLI, as described [here](../Getting-started/log-in-to-openshift.md).

A set of helpful common `oc` commands can be found [here](../Getting-started/oc-commands.md).

Once you're logged in, create a new project for this deployment.

```
$ oc new-project userXX-lab02-nodejs
```

Replace `userXX` with your user ID or other name.

Create a new local directory for this lab, and change to it.

```
$ mkdir Lab02
$ cd Lab02
```

One of the first things to think about is how we're going to store some credentials. Secrets offer a way to store encrypted sensitive application data within the OpenShift platform.

Create a file called `ws-secret.yaml`
​
```
cat <<EOF  > ws-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: ws-secret
data:
  username: YWRtaW4K
  password: cGFzc3dvcmQ=
EOF
```

​
ConfigMap provide us with a way to store non-sensitive information, such as environment variables or configuration files.
​
Create a file called `ws-config.yaml` with the following data
​
```
cat <<EOF > ws-cm.yaml
kind: ConfigMap 
apiVersion: v1 
metadata:
  name: ws-config
data:
  # Configuration values can be set as key-value properties
  MONGODB: mongodb
  MONGODB_URI: mongodb://localhost:27017
EOF
```
​
​
Now, we'll set up a `deployment.yaml` file to bring the Deployment specification, ConfigMap and Secret together.

Create a file called `ws-deployment.yaml` with the following data
​
```
cat <<EOF > ws-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: websphere-liberty-app
spec:
  selector:
    matchLabels:
      app: websphere-liberty-app
  replicas: 1
  template:
    metadata:
      labels:
        app: websphere-liberty-app
    spec:
      containers:
        - name: websphere-liberty-app
          image: websphere-liberty:19.0.0.9-webProfile8-java11
          ports:
            - containerPort: 9080
          volumeMounts:
            - mountPath: "/config/dropins"
              name: was-persistence
          envFrom:
            - configMapRef:
                name: ws-config
            - secretRef:
                name: ws-secret
      volumes:
        - name: was-persistence
          emptyDir: {}
EOF
```
​
​
Create the resources in OpenShift with the `oc create` command. You can do this on an entire directory too.
```
$ oc create -f .
configmap/ws-config created
deployment.apps/websphere-liberty-app created
secret/ws-secret created
```
​
You should now be able to use `oc get` commands to check the resources in the OpenShift cluster. For example, to get all deployments in the current project, use `oc get deployment`. To get all pods in the project, use `oc get pods`. These are very frequently used commands and have many variations through the use of filters and selectors, which is useful in clusters that have many applications per project.

```
$ oc get deploy
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
websphere-liberty-app   0/1     1            0           17s
...
$ oc get pods
NAME                                     READY   STATUS              RESTARTS   AGE
websphere-liberty-app-556bdf88b6-bzn9x   0/1     ContainerCreating   0          15s
```

After a short time, the deployment should have 1/1 READY and the pod should be running.

```
$ oc get deploy
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
websphere-liberty-app   1/1     1            1           73s

$ oc get pods
NAME                                     READY   STATUS    RESTARTS   AGE
websphere-liberty-app-556bdf88b6-bzn9x   1/1     Running   0          71s
```
​
To check that the WebSphere pod has consumed the data from the ConfigMap, we can use `oc rsh` to open a remote shell session directly to the container. This is a similar action to the `docker exec -it` command run in previous labs.
​
```
$ oc rsh <pod_name>
```
​
Check the environment variables present in the container
```
$ echo $MONGODB
$ echo $MONGODB_URI
$ echo $password
$ echo $username
```
​
So now that the WebSphere application is deployed, we need to expose it so we can access the application from outside OpenShift. To do this, we create two resources

1. A Service
2. A Route to the Service
​
​
The Service will expose the specified container port and provide it with an IP and DNS name in the Service IP range. In this instance, we only have one pod but Services become more important when you have multiple replicas of an application and you expect the traffic sent to mulitple replicas to be properly load balanced and received by all relevant pods.

Create a file called `ws-svc.yaml` with the following content
​
```
cat <<EOF > ws-svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: websphere-liberty
spec:
  ports:
  - port: 9080
  selector:
    app: websphere-liberty-app
EOF
```
​
Create the Service in OpenShift
```
oc create -f ws-svc.yaml
service/websphere-liberty created
```

View that the service was created using `oc get svc`

```
$ oc get svc websphere-liberty
NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
websphere-liberty   ClusterIP   172.30.51.229   <none>        9080/TCP   12s
```
​
Notice that the Service created shows as a `ClusterIP` type. This is the default if we do not specify a specific method in the `spec.type` field. In this lab we will expose this Service through the OpenShift Router using an OpenShift Route, but in other cases you may want to expose the applcation using a different method, such as NodePort.

To easily expose the application, use the following command

```
$ oc expose service websphere-liberty
route.route.openshift.io/websphere-liberty exposed
```

You can view the new Route using the following command

```
oc get routes
NAME                HOST/PORT                                              PATH   SERVICES            PORT   TERMINATION   WILDCARD
websphere-liberty   websphere-liberty-was-test.apps.ocp4.os.fyre.ibm.com          websphere-liberty   9080                 None
```

In your web browser, navigate to the URL given in the HOST/PORT section. This should bring up the WebSphere landing page.

Lab complete.