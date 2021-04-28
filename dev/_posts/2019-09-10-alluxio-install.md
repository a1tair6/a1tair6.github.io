# alluxio 설치
- data lake 구축
- aws s3, gcp storage, hdfs 통합 구축 할 수 있는 open source

## rancher config maps
```
ALLUXIO_MASTER_HOSTNAME		alluxio-master.alluxio
ALLUXIO_MASTER_JAVA_OPTS		-Dalluxio.master.security.impersonation.hadoop.users=* -Dalluxio.master.mount.table.root.option.aws.accessKeyId={accesskey} -Dalluxio.master.mount.table.root.option.aws.secretKey={secretKey}
ALLUXIO_MASTER_MOUNT_TABLE_ROOT_OPTION_ALLUXIO_UNDERFS_VERSION		3.1
ALLUXIO_MASTER_MOUNT_TABLE_ROOT_UFS		hdfs://{hostname}:{port}
ALLUXIO_WORKER_MEMORY_SIZE		1GB
S3_ENDPOINT		s3.{region}.amazonaws.com
```

## alluxio-master yaml
- rancher workloads에서 import yaml
```yml
apiVersion: v1
kind: Service
metadata:
  name: alluxio-master
  labels:
    app: alluxio
spec:
  ports:
  - port: 19998
    name: rpc
  - port: 19999
    name: web
  clusterIP: None
  selector:
    app: alluxio-master
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: alluxio-pv-claim
spec:
  storageClassName: gp2
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: alluxio-master
spec:
  selector:
    matchLabels:
      app: alluxio-master
  serviceName: "alluxio-master"
  replicas: 1
  template:
    metadata:
      labels:
        app: alluxio-master
    spec:
      hostNetwork: true
      dnsPolicy: ClusterFirstWithHostNet
      containers:
        - name: alluxio-master
          image: alluxio/alluxio:2.0.0
          resources:
            requests:
              cpu: "0.5"
              memory: "512M"
            limits:
              cpu: "1"
              memory: "1024M"
          command: ["/entrypoint.sh"]
          args: ["master-only"]
          envFrom:
          - configMapRef:
              name: alluxio-config
          ports:
          - containerPort: 19998
            name: rpc
          - containerPort: 19999
            name: web
                     volumeMounts:
            - name: alluxio-journal
              mountPath: /journal
        - name: alluxio-job-master
          image: alluxio/alluxio:2.0.0
          resources:
            requests:
              cpu: "0.5"
              memory: "512M"
            limits:
              cpu: "1"
              memory: "1024M"
          command: ["/entrypoint.sh"]
          args: ["job-master"]
          envFrom:
          - configMapRef:
              name: alluxio-config
          ports:
          - containerPort: 20001
            name: job-rpc
          - containerPort: 20002
            name: job-web
      hostAliases:
      - hostnames:
        - {hostname}
        ip: {ip}

      restartPolicy: Always
      volumes:
        - name: alluxio-journal
          persistentVolumeClaim:
            claimName: alluxio-pv-claim
```

## alluxio-worker yaml
```yml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: alluxio-worker
spec:
  selector:
    matchLabels:
      name: alluxio-worker
  template:
    metadata:
      labels:
        name: alluxio-worker
        app: alluxio
    spec:
      hostNetwork: true
      dnsPolicy: ClusterFirstWithHostNet
      containers:
        - name: alluxio-worker
          image: alluxio/alluxio:2.0.0
          resources:
            requests:
              cpu: "0.5"
              memory: "2G"
            limits:
              cpu: "1"
              memory: "2G"
          command: ["/entrypoint.sh"]
          args: ["worker-only", "--no-format"]
          env:
          - name: ALLUXIO_WORKER_HOSTNAME
            valueFrom:
              fieldRef:
                fieldPath: status.hostIP
          - name: ALLUXIO_WORKER_JAVA_OPTS
            value: " -Dalluxio.worker.hostname=$(ALLUXIO_WORKER_HOSTNAME) "
          envFrom:
          - configMapRef:
              name: alluxio-config
          ports:
          - containerPort: 29998
            name: rpc
          - containerPort: 29999
            name: data
          - containerPort: 29996
            name: web
          volumeMounts:
            - name: alluxio-ramdisk
              mountPath: /dev/shm
            - name: alluxio-domain
              mountPath: /opt/domain
        - name: alluxio-job-worker
          image: alluxio/alluxio:2.0.0
          resources:
            requests:
              cpu: "0.5"
              memory: "2G"
            limits:
              cpu: "1"
              memory: "2G"
          command: ["/entrypoint.sh"]
          args: ["job-worker"]
          env:
          - name: ALLUXIO_WORKER_HOSTNAME
            valueFrom:
              fieldRef:
                              fieldPath: status.hostIP
          - name: ALLUXIO_WORKER_JAVA_OPTS
            value: " -Dalluxio.worker.hostname=$(ALLUXIO_WORKER_HOSTNAME) "
          envFrom:
          - configMapRef:
              name: alluxio-config
          ports:
          - containerPort: 30001
            name: job-rpc
          - containerPort: 30002
            name: job-data
          - containerPort: 30003
            name: job-web
          volumeMounts:
            - name: alluxio-ramdisk
              mountPath: /dev/shm
            - name: alluxio-domain
              mountPath: /opt/domain
      hostAliases:
      - hostnames:
        - {hostname}
        ip: {ip}
      restartPolicy: Always
      volumes:
        - name: alluxio-ramdisk
          emptyDir:
            medium: "Memory"
            sizeLimit: "1G"
        - name: alluxio-domain
          hostPath:
            path: /tmp/domain
            type: DirectoryOrCreate
```
