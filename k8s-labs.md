# Kubernetes labs for CKAD

1. Create a pod with name **redis** with image **k8s.gcr.io/redis** with config property **maxmemory**=2097152 and **maxmemory-policy**=allkeys-lru. Mount the data folder to **redis-master-data** path and config to **redis-master**

   > *redis-configmap.yaml*

   ```yaml
   ---
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: redis-configmap
   data:
     redis-config: |
       maxmemory 2mb
       maxmemory-policy allkeys-lru
   ```

   > *pod-redis.yaml*

   ```yaml
   ---
   apiVersion: v1
   kind: Pod
   metadata:
     name: redis
   spec:
     containers:
     - name: redis
       image: redis
       volumeMounts:
       - name: redis-config-vol
         mountPath: /redis-master
       - name: redis-data-vol
         mountPath: /redis-master-data
     volumes:
     - name: redis-config-vol
       configMap:
         name: redis-configmap
     - name: redis-data-vol
       emptyDir: {}
   ```

2. Create a pod with environment variable **LOG_LEVEL** using configmap property **log_level=INFO**. Use busybox image

   > *cm-log-level.yaml*

   ```yaml
   ---
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: log-level-config
   data:
     log_level: INFO
   ```

   > *pod-env-configmap.yaml*

   ```yaml
   ---
   apiVersion: v1
   kind: Pod
   metadata:
     name: busybox-envconfig
   spec:
     containers:
     - name: busybox
       image: busybox
       args: [/bin/sh, -c, env]
       env:
       - name: LOG_LEVEL
         valueFrom:
           configMapKeyRef:
             name: log-level-config
             key: log_level
   ```

3. Create a multicontainer pod using **2 sidecar containers** to tail the log file written by the logging container

   > *pod-logging-sidecars.yaml*

   ```yaml
   ---
   apiVersion: v1
   kind: Pod
   metadata:
     name: logging-sidecars
     labels:
       app: logging
   spec:
     containers:
     - name: busybox-main
       image: busybox
       args:
       - /bin/sh
       - -c
       - >
         i=0;
           while true;
           do
             echo "$i: $(date)" >> /var/log/1.log;
             echo "$(date) INFO $i" >> /var/log/2.log;
             i=$((i+1));
             sleep 1;
           done
       volumeMounts:
       - name: logvol
         mountPath: /var/log
     - name: busybox-sidecar1
       image: busybox
       args: [/bin/sh, -c, tail -n+1 -f /var/log/1.log]
       volumeMounts:
       - name: logvol
         mountPath: /var/log
     - name: busybox-sidecar2
       image: busybox
       args: [/bin/sh, -c, tail -n+1 -f /var/log/2.log]
       volumeMounts:
       - name: logvol
         mountPath: /var/log
     volumes:
     - name: logvol
       emptyDir: {}
   ```

4. Create multicontainer pod with fluentd to collect logs written by the main container to /var/logs/1.log and var/logs/2.log. Use image k8s.gcr.io/fluentd-gcp:1.30 for fluentd container and busybox for main container

   > *fluentd-configmap.yaml*

   ```yaml
   ---
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: fluentd-configmap
   data:
     fluentd.conf: |
       <source>
         type tail
         format none
         path /var/log/1.log
         pos_file /var/log/1.log.pos
         tag count.format1
       </source>
       <source>
         type tail
         format none
         path /var/log/2.log
         pos_file /var/log/2.log.pos
         tag count.format2
       </source>
       <match **>
         type file
         path /var/log/syslog
       </match>
   ```

   > *pod-fluentd.yaml*

   ```yaml
   ---
   apiVersion: v1
   kind: Pod
   metadata:
     name: busybox-logging
   spec:
     containers:
       - name: busybox
         image: busybox
         args:
         - /bin/sh
         - -c
         - >
            i=0;
           while true;
           do
             echo "$i: $(date)" >> /var/log/1.log;
             echo "$(date) INFO $i" >> /var/log/2.log;
             i=$((i+1));
             sleep 1;
           done
         volumeMounts:
         - name: varlog
           mountPath: /var/logs
       - name: fluentd-logging-agent
         image: k8s.gcr.io/fluentd-gcp:1.30
         env:
         - name: FLUENTD_ARGS
           value: -c /etc/fluentd-config/fluentd.conf
         volumeMounts:
         - name: fluentd-config-vol
           mountPath: /etc/fluentd-config
     volumes:
     - name: varlog
       emptyDir: {}
     - name: fluentd-config-vol
     	configMap:
     	  name: fluentd-configmap
   ```

5. Create a nginx deployment with 3 replicas

   > *deployment-nginx.yaml*

   ```
   
   ```

   

6. 