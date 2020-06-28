# Exercise 2 - Creating a Cassandra StatefulSet with Dynamic Storage

In this lab you'll create a new Cassandra database application as a that will leverage the storage class created in previous labs.

To get started, log into OpenShift using the CLI, as described [here](../Getting-started/log-in-to-openshift.md).

A set of helpful common `oc` commands can be found [here](../Getting-started/oc-commands.md).

Once you're logged in, create a new project for this deployment.

```
$ oc new-project userXX-lab05-dynamic-pvc-sts
```

Replace `userXX` with your user ID or other name.

Create a new local directory for this lab, and change to it.

```
$ mkdir -p Lab05/dynamic-pvc-sts
$ cd Lab05/dynamic-pvc-sts
```

Like a Deployment, a StatefulSet manages Pods that are based on an identical container spec. Unlike a Deployment, a StatefulSet maintains a sticky identity for each of their Pods. These pods are created from the same spec, but are not interchangeable: each has a persistent identifier that it maintains across any rescheduling.

This example will show how a StatefulSet can be coupled with dynamic storage provisioning to fully utilise scalable stateful container applications.

First, we'll create something called a Headless Service. A headless service is a service with a service IP but instead of load-balancing it will return the IPs of our associated Pods. This allows us to interact directly with the Pods instead of a proxy. It's as simple as specifying None for `.spec.clusterIP` and can be utilized with or without selectors. This becomes useful when you want to divert traffic to the individual addresses of each database pod as opposed to letting the service load balance across all pods assigned to the service. For example, for a high-availability database deployment where the leading database needs to keep track of the other cluster members and send data directly to each member pod, using a normal Service will not suffice as the data will be routed to a randomly selected load-balanced pod instead of the target. 

To create the headless service, create a file calledd `headless-svc.yaml`

```
cat <<EOF > headless-svc.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: cassandra
  name: cassandra
spec:
  clusterIP: None
  ports:
  - port: 9042
  selector:
    app: cassandra
EOF
```

Create the service

```
$ oc create -f headless-svc.yaml

```

Next we need to deploy the StatefulSet. Below is the definition for a Cassandra database with 3 replicas. With the use of the `volumeClaimTemplates`, we can define specific storage requirements for the database pods. It also means that each pod in the StatefulSet will create it's own indexed Persistent Volume Claim using the dynamic provisioner we created earlier. 

Use the following definition as a template, changing the following values

Change the `CASSANDRA_SEEDS` environment variable to match your project name. For example, for the `user99-lab05-dynamic-pvc-sts` project, change the value from this
```
- name: CASSANDRA_SEEDS
  value: "cassandra-0.cassandra.default.svc.cluster.local"
```
to this
```
- name: CASSANDRA_SEEDS
  value: "c"
```

The format of the above service is `cassandra-<index>.<sts-name>.project.cluster.local`

Create a new file `cassandra-sts.yaml`

```
cat <<EOF > cassandra-sts.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: cassandra
  labels:
    app: cassandra
spec:
  serviceName: cassandra
  replicas: 3
  selector:
    matchLabels:
      app: cassandra
  template:
    metadata:
      labels:
        app: cassandra
    spec:
      terminationGracePeriodSeconds: 1800
      containers:
      - name: cassandra
        image: gcr.io/google-samples/cassandra:v13
        imagePullPolicy: Always
        ports:
        - containerPort: 7000
          name: intra-node
        - containerPort: 7001
          name: tls-intra-node
        - containerPort: 7199
          name: jmx
        - containerPort: 9042
          name: cql
        resources:
          limits:
            cpu: "500m"
            memory: 1Gi
          requests:
            cpu: "500m"
            memory: 1Gi
        securityContext:
          capabilities:
            add:
              - IPC_LOCK
        lifecycle:
          preStop:
            exec:
              command: 
              - /bin/sh
              - -c
              - nodetool drain
        env:
          - name: MAX_HEAP_SIZE
            value: 512M
          - name: HEAP_NEWSIZE
            value: 100M
          - name: CASSANDRA_SEEDS
            value: "cassandra-0.cassandra.user99-lab05-dynamic-pvc-sts.svc.cluster.local"
          - name: CASSANDRA_CLUSTER_NAME
            value: "K8Demo"
          - name: CASSANDRA_DC
            value: "DC1-K8Demo"
          - name: CASSANDRA_RACK
            value: "Rack1-K8Demo"
          - name: POD_IP
            valueFrom:
              fieldRef:
                fieldPath: status.podIP
        readinessProbe:
          exec:
            command:
            - /bin/bash
            - -c
            - /ready-probe.sh
          initialDelaySeconds: 15
          timeoutSeconds: 5
        volumeMounts:
        - name: cassandra-data
          mountPath: /cassandra_data
  volumeClaimTemplates:
  - metadata:
      name: cassandra-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: user99-dynamic-nfs-storage
      resources:
        requests:
          storage: 1Gi
EOF
```

Verify the Statefulset was created

```
$ oc get statefulset
NAME        READY   AGE
cassandra   0/3     3s

$ oc get pods

```