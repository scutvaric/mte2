# Kubernetes Exam — Submission Sheet (Q1–Q53)

For each question: **Commands** to run, a short **first-person explanation** to write in the submission, and the **Proof** (output) to capture as evidence.

---

# LO4 — Accelerated Delivery Using Containers

## Q1: Deployment `web`, nginx:1.25, 3 replicas, export YAML

```bash
kubectl create deployment web --image=docker.io/library/nginx:1.25 --replicas=3
kubectl get deployment web -o yaml > web-deployment.yaml
```

**Explanation:** I created a Deployment called web running nginx:1.25 with three replicas, then exported its live definition to a YAML file so I could edit or reapply it later.

**Proof:** `kubectl get deployment web` showing READY 3/3, and the existence of `web-deployment.yaml`.

---

## Q2: Authenticated DockerHub pulls — what resource?

```bash
kubectl create secret docker-registry dockerhub-secret --docker-username=YOUR_USERNAME --docker-password=YOUR_PASSWORD --docker-email=YOUR_EMAIL
```

```yaml
spec:
  imagePullSecrets:
  - name: dockerhub-secret
```

**Explanation:** To pull as an authenticated DockerHub user I need an image-pull Secret of type docker-registry, which stores my credentials. I reference it under imagePullSecrets in the pod spec so Kubernetes authenticates when pulling and avoids the anonymous rate limit.

**Proof:** `kubectl get secret dockerhub-secret` showing type kubernetes.io/dockerconfigjson.

---

## Q3: Scale 3→5 two ways

```bash
kubectl scale deployment web --replicas=5
kubectl edit deployment web      # change replicas: 3 to 5, save with :wq
kubectl get replicaset
```

**Explanation:** I scaled the deployment imperatively with kubectl scale, and declaratively by editing the manifest. Scaling changes the replica count on the same ReplicaSet — a new ReplicaSet is only created when the pod template changes, not when scaling.

**Proof:** `kubectl get pods -l app=web` showing 5 pods; `kubectl get replicaset` showing the count rise to 5.

---

## Q4: Rolling update 1.25→1.27, watch rollout

```bash
kubectl set image deployment/web nginx=docker.io/library/nginx:1.27
kubectl rollout status deployment/web
```

**Explanation:** I performed a rolling update to nginx:1.27. A rolling update replaces pods gradually, keeping the app available the whole time, and rollout status let me watch the new replicas come up.

**Proof:** rollout status output ending in "successfully rolled out"; `kubectl describe deployment web | grep Image` showing 1.27.

---

## Q5: Rollout history + rollback, explain CHANGE-CAUSE

```bash
kubectl rollout history deployment/web
kubectl annotate deployment/web kubernetes.io/change-cause='Updated to nginx 1.27'
kubectl rollout undo deployment/web
kubectl rollout undo deployment/web --to-revision=1
```

**Explanation:** I viewed the revision history and rolled back to the previous revision. CHANGE-CAUSE is a human-readable note shown against each revision explaining what changed, which I set with an annotation.

**Proof:** `kubectl rollout history deployment/web` showing revisions with the CHANGE-CAUSE text, and the image reverting after undo.

---

## Q6: RollingUpdate maxSurge:1, maxUnavailable:0

```bash
kubectl patch deployment web -p '{"spec":{"strategy":{"type":"RollingUpdate","rollingUpdate":{"maxSurge":1,"maxUnavailable":0}}}}'
```

**Explanation:** maxUnavailable:0 means no pod is taken down before a replacement is Ready, so I always keep the full count running — zero downtime. maxSurge:1 lets one extra pod start during the update, so I briefly run N+1 pods.

**Proof:** `kubectl describe deployment web | grep -A2 RollingUpdateStrategy` showing 1 max surge, 0 max unavailable.

---

## Q7: Switch to Recreate, when required

```bash
kubectl patch deployment web -p '{"spec":{"strategy":{"type":"Recreate"}}}'
```

**Explanation:** Recreate terminates all old pods first, then starts new ones, which causes downtime. It's required when old and new versions can't run at the same time — for example an incompatible database schema change.

**Proof:** `kubectl describe deployment web | grep StrategyType` showing Recreate.

---

## Q8: Add requests/limits, verify in pod

```bash
kubectl set resources deployment/web --requests=cpu=100m,memory=128Mi --limits=cpu=500m,memory=256Mi
kubectl get pod -l app=web -o jsonpath='{.items[0].spec.containers[0].resources}'
```

**Explanation:** I set CPU/memory requests (guaranteed minimum, used for scheduling) and limits (hard ceiling). Without them a pod could consume all node resources. 100m is a tenth of a core; 128Mi is 128 mebibytes.

**Proof:** the jsonpath output showing the requests and limits on the running pod.

---

## Q9: revisionHistoryLimit:3

```bash
kubectl patch deployment web -p '{"spec":{"revisionHistoryLimit":3}}'
```

**Explanation:** This keeps only the last three old ReplicaSets, so I can roll back at most three revisions; older ones are pruned to save space. The default is 10.

**Proof:** `kubectl get deployment web -o jsonpath='{.spec.revisionHistoryLimit}'` showing 3.

---

## Q10: Bad image tag → stuck rollout, old pods keep serving

```bash
kubectl set image deployment/web nginx=docker.io/library/nginx:doesnotexist999
kubectl rollout status deployment/web
kubectl get pods -l app=web
kubectl rollout undo deployment/web
```

**Explanation:** With a non-existent tag the new pods can't pull the image and sit in ImagePullBackOff, so the rollout stalls. Because maxUnavailable holds the old pods until new ones are Ready — which never happens — the old pods keep serving and users see no disruption. I rolled back to fix it.

**Proof:** `kubectl get pods` showing new pods in ImagePullBackOff while old pods stay Running; rollout recovering after undo.

---

## Q11: Label selector to list pods; Deployment→ReplicaSet→Pod

```bash
kubectl get pods -l app=web
kubectl get replicaset -l app=web
```

**Explanation:** A Deployment manages ReplicaSets, a ReplicaSet manages Pods, and labels tie them together. The label selector app=web lets me list exactly the pods belonging to this deployment. Changing the pod template makes the Deployment spin up a new ReplicaSet.

**Proof:** both commands listing the matching ReplicaSet and pods.

---

## Q12: Expose deployment, explain Service + selector

```bash
kubectl expose deployment web --port=80 --target-port=80
kubectl describe service web
```

**Explanation:** kubectl expose created a Service (ClusterIP by default). Its selector was copied automatically from the deployment's pod labels (app=web), and the Service uses that selector to find which pods to route traffic to.

**Proof:** `kubectl describe service web` showing the Selector app=web and the endpoints.

---

## Q13: Add a sidecar; shared network/volumes

```bash
kubectl edit deployment web
```

```yaml
      volumes:
      - name: logs
        emptyDir: {}
      containers:
      - name: nginx
        image: docker.io/library/nginx:1.25
        volumeMounts:
        - name: logs
          mountPath: /var/log/nginx
      - name: log-sidecar
        image: docker.io/library/busybox:latest
        command: ['sh','-c','tail -f /var/log/nginx/access.log']
        volumeMounts:
        - name: logs
          mountPath: /var/log/nginx
```

**Explanation:** I added a second (sidecar) container that tails nginx's logs. All containers in a pod share one network namespace (same IP/localhost) and can share volumes, so the sidecar reaches nginx at localhost:80 and reads its logs through a shared emptyDir volume mounted by both containers.

**Proof:** `kubectl get pods -l app=web` showing 2/2 containers ready; `kubectl logs <pod> -c log-sidecar` showing log output.

---

## Q14: httpd:2.4 deployment, 2 replicas, named port, nodeSelector; explain Pending

```bash
nano httpd-deployment.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: httpd-app
  template:
    metadata:
      labels:
        app: httpd-app
    spec:
      nodeSelector:
        disktype: ssd
      containers:
      - name: httpd
        image: docker.io/library/httpd:2.4
        ports:
        - name: http
          containerPort: 80
```

```bash
kubectl apply -f httpd-deployment.yaml
kubectl get pods -l app=httpd-app
```

**Explanation:** I wrote a Deployment for httpd:2.4 with two replicas, a named port (http/80), and a nodeSelector requiring disktype=ssd. The pods stay Pending because no node carries that label, so the scheduler can't place them. Labelling a node disktype=ssd would let them schedule.

**Proof:** `kubectl get pods -l app=httpd-app` showing Pending; `kubectl describe pod <name>` showing the "node(s) didn't match node selector" event.

---

## Q15: Bare Pod vs Deployment-managed pod on delete

```bash
kubectl run bare-pod --image=docker.io/library/busybox:latest --restart=Never -- sleep 3600
kubectl delete pod bare-pod
kubectl delete pod <web-pod-name>
```

**Explanation:** A bare pod has no controller, so deleting it removes it permanently. A Deployment-managed pod is recreated immediately by its ReplicaSet, because the controller maintains the desired replica count.

**Proof:** after deleting bare-pod, `kubectl get pods` shows it gone; after deleting a web pod, a new one appears.

---

## Q16: StatefulSet redis:7, 3 replicas + headless Service; pod names + DNS

```bash
nano redis-statefulset.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis
spec:
  clusterIP: None
  selector:
    app: redis
  ports:
  - port: 6379
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
spec:
  serviceName: redis
  replicas: 3
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: docker.io/library/redis:7
        ports:
        - containerPort: 6379
```

```bash
kubectl apply -f redis-statefulset.yaml
kubectl get pods -l app=redis
kubectl get statefulset
```

**Explanation:** I created a StatefulSet of three redis pods plus a headless Service (clusterIP: None). The StatefulSet gives pods stable ordinal names redis-0/1/2, and the headless service gives each a stable DNS record of the form pod.service.namespace.svc.cluster.local, e.g. redis-0.redis.default.svc.cluster.local.

**Proof:** `kubectl get pods -l app=redis` showing redis-0, redis-1, redis-2; `kubectl get service redis` showing CLUSTER-IP None.

---

## Q17: DaemonSet busybox; one pod per node

```bash
nano busybox-daemonset.yaml
```

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: busybox-agent
spec:
  selector:
    matchLabels:
      app: busybox-agent
  template:
    metadata:
      labels:
        app: busybox-agent
    spec:
      containers:
      - name: agent
        image: docker.io/library/busybox:latest
        command: ['sh','-c','while true; do echo running; sleep 60; done']
```

```bash
kubectl apply -f busybox-daemonset.yaml
kubectl get daemonset
kubectl get pods -l app=busybox-agent -o wide
```

**Explanation:** A DaemonSet runs exactly one pod on every eligible node and automatically adds one when a new node joins — useful for per-node agents like log or monitoring collectors. On this single-node lab that means one pod.

**Proof:** `kubectl get pods -l app=busybox-agent -o wide` showing one pod per node.

---

## Q18: Job busybox computes once; COMPLETIONS + logs

```bash
kubectl create job compute-job --image=docker.io/library/busybox:latest -- /bin/sh -c 'echo Result: $((2+2))'
kubectl get jobs
kubectl logs job/compute-job
```

**Explanation:** A Job runs a one-off task to completion. This one opens a shell, computes 2+2, and prints the result, then the Job is marked Complete.

**Proof:** `kubectl get jobs` showing COMPLETIONS 1/1; `kubectl logs job/compute-job` showing "Result: 4".

---

## Q19: CronJob prints date every minute; suspend; list spawned Jobs

```bash
kubectl create cronjob date-printer --image=docker.io/library/busybox:latest --schedule='*/1 * * * *' -- date
kubectl patch cronjob date-printer -p '{"spec":{"suspend":true}}'
kubectl get jobs
```

**Explanation:** A CronJob runs a Job on a schedule (here every minute, cron */1 * * * *). Suspending it stops future runs without deleting it. Each run creates a Job named date-printer-<timestamp>.

**Proof:** `kubectl get cronjob` showing SUSPEND True; `kubectl get jobs` listing the spawned date-printer-* jobs.

---

## Q20: Manually run a Job from a CronJob

```bash
kubectl create job test-run --from=cronjob/date-printer
kubectl logs job/test-run
```

**Explanation:** I triggered an on-demand run from the existing CronJob with --from, which is handy for testing without waiting for the schedule.

**Proof:** `kubectl get jobs` showing test-run Complete; `kubectl logs job/test-run` showing the date output.

---

## Q21: StatefulSet ordered create/terminate vs Deployment

```bash
# terminal 1:
kubectl get pods -w
# terminal 2:
kubectl delete statefulset redis
kubectl apply -f redis-statefulset.yaml
```

**Explanation:** A StatefulSet creates pods in order 0→1→2, waiting for each to be Ready before the next, and terminates them in reverse 2→1→0. A Deployment starts and stops all pods together with no guaranteed order. Ordering matters for stateful apps where a primary must be ready before replicas connect.

**Proof:** the watch output showing redis-0 Ready before redis-1 starts, etc.

---

## Q22: Multi-container pod sharing emptyDir, writer+reader

```bash
nano multi-container.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: shared-volume-pod
spec:
  volumes:
  - name: shared
    emptyDir: {}
  containers:
  - name: writer
    image: docker.io/library/busybox:latest
    command: ['sh','-c','echo hello > /shared/message.txt && sleep 3600']
    volumeMounts:
    - name: shared
      mountPath: /shared
  - name: reader
    image: docker.io/library/busybox:latest
    command: ['sh','-c','sleep 5 && cat /shared/message.txt && sleep 3600']
    volumeMounts:
    - name: shared
      mountPath: /shared
```

```bash
kubectl apply -f multi-container.yaml
kubectl logs shared-volume-pod -c reader
```

**Explanation:** Two containers in one pod share an emptyDir volume. The writer creates a file in it and the reader reads the same file, proving the volume is shared between containers.

**Proof:** `kubectl logs shared-volume-pod -c reader` showing "hello".

---

## Q23: List StorageClasses, PVs, PVCs

```bash
kubectl get storageclass
kubectl get pv
kubectl get pvc
kubectl get pvc -A
```

**Explanation:** StorageClasses define how storage is provisioned, PersistentVolumes are the actual storage pieces, and PersistentVolumeClaims are requests pods make for storage. I listed each, including PVCs across all namespaces.

**Proof:** the output of the four commands.

---

## Q24: Mount ConfigMap as volume, each key a file

```bash
kubectl create configmap app-config --from-literal=database.host=mysql --from-literal=database.port=3306
nano config.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-pod
spec:
  volumes:
  - name: config
    configMap:
      name: app-config
  containers:
  - name: app
    image: docker.io/library/busybox:latest
    command: ['sleep','3600']
    volumeMounts:
    - name: config
      mountPath: /etc/config
```

```bash
kubectl apply -f config.yaml
kubectl exec config-pod -- ls /etc/config
kubectl exec config-pod -- cat /etc/config/database.host
```

**Explanation:** Mounting a ConfigMap as a volume turns each key into a file under the mount path. So database.host and database.port become files I can read inside the pod.

**Proof:** `ls /etc/config` showing database.host and database.port; `cat /etc/config/database.host` showing mysql.

---

## Q25: subPath single key to a path; when needed + update caveat

```bash
nano config.yaml      # change the volumeMounts as below
```

```yaml
    volumeMounts:
    - name: config
      mountPath: /etc/config/database.host
      subPath: database.host
```

```bash
kubectl delete pod config-pod
kubectl apply -f config.yaml
kubectl exec config-pod -- ls /etc/config/
kubectl exec config-pod -- cat /etc/config/database.host
```

**Explanation:** subPath mounts just one key to a specific file path instead of replacing the whole directory — needed when a directory already has other files I want to keep. The caveat is that subPath-mounted files do not auto-update when the ConfigMap changes; the pod must be restarted, whereas non-subPath mounts update eventually.

**Proof:** `ls /etc/config/` showing only the single file; `cat` showing mysql.

---

## Q26: Mount Secret as volume, decoded values + restrictive perms

```bash
kubectl create secret generic db-creds --from-literal=username=admin --from-literal=password=secret123
nano secret-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  volumes:
  - name: creds
    secret:
      secretName: db-creds
      defaultMode: 0400
  containers:
  - name: app
    image: docker.io/library/busybox:latest
    command: ['sleep','3600']
    volumeMounts:
    - name: creds
      mountPath: /etc/secrets
```

```bash
kubectl apply -f secret-pod.yaml
kubectl exec secret-pod -- cat /etc/secrets/username
kubectl exec secret-pod -- ls -laL /etc/secrets
```

**Explanation:** Mounting a Secret as a volume presents each key as a file holding the decoded (plaintext) value, and defaultMode 0400 makes the files read-only to the owner. So the mounted username file reads "admin", not the base64 form.

**Proof:** `cat /etc/secrets/username` showing admin; `ls -laL /etc/secrets` showing -r-------- permissions.

---

# LO (Storage / Secrets / Services / Probes) — Q27–Q53

## Q27: StatefulSet PVC per replica; what happens on delete

```bash
kubectl delete statefulset redis
nano redis-statefulset.yaml      # add volumeClaimTemplates (see below)
```

```yaml
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 100Mi
```

```bash
kubectl apply -f redis-statefulset.yaml
kubectl get pvc
kubectl delete statefulset redis
kubectl get pvc        # PVCs still exist
```

**Explanation:** A StatefulSet's volumeClaimTemplates create one PVC per pod automatically. Deleting the StatefulSet does NOT delete the PVCs — the data is preserved on purpose, so I must delete PVCs manually if I want the storage gone.

**Proof:** `kubectl get pvc` showing data-redis-0/1/2 before and still present after the StatefulSet is deleted.

---

## Q28: initContainer pre-populates shared volume

```bash
nano init-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-pod
spec:
  volumes:
  - name: shared
    emptyDir: {}
  initContainers:
  - name: setup
    image: docker.io/library/busybox:latest
    command: ['sh','-c','echo pre-loaded data > /data/file.txt']
    volumeMounts:
    - name: shared
      mountPath: /data
  containers:
  - name: main
    image: docker.io/library/busybox:latest
    command: ['sh','-c','cat /data/file.txt && sleep 3600']
    volumeMounts:
    - name: shared
      mountPath: /data
```

```bash
kubectl apply -f init-pod.yaml
kubectl exec init-pod -- cat /data/file.txt
kubectl logs init-pod -c main
```

**Explanation:** The initContainer runs to completion before the main container starts, writing a file into a shared emptyDir volume. The main container then reads that pre-populated file, proving the init step ran first.

**Proof:** `kubectl logs init-pod -c main` showing "pre-loaded data"; `kubectl describe pod init-pod` showing the init container Terminated/Completed.

---

## Q29: readOnly:true mount, prove writes rejected

```bash
nano readonly-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readonly-pod
spec:
  volumes:
  - name: config
    configMap:
      name: app-config
  containers:
  - name: app
    image: docker.io/library/busybox:latest
    command: ['sleep','3600']
    volumeMounts:
    - name: config
      mountPath: /etc/config
      readOnly: true
```

```bash
kubectl apply -f readonly-pod.yaml
kubectl exec readonly-pod -- touch /etc/config/test
kubectl exec readonly-pod -- cat /etc/config/database.host
```

**Explanation:** Setting readOnly: true makes the mount read-only, so any write is rejected by the filesystem while reads still work. (Requires the app-config ConfigMap from Q24 to exist.)

**Proof:** the touch failing with "Read-only file system"; the cat returning mysql.

---

## Q30: emptyDir sizeLimit; exceeding it

```bash
nano sizelimit-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sizelimit-pod
spec:
  volumes:
  - name: temp
    emptyDir:
      sizeLimit: 100Mi
  containers:
  - name: app
    image: docker.io/library/busybox:latest
    command: ['sleep','3600']
    volumeMounts:
    - name: temp
      mountPath: /tmp/data
```

```bash
kubectl apply -f sizelimit-pod.yaml
kubectl describe pod sizelimit-pod        # shows SizeLimit: 100Mi
kubectl exec sizelimit-pod -- dd if=/dev/zero of=/tmp/data/bigfile bs=1M count=110
kubectl get pods sizelimit-pod            # Evicted
```

**Explanation:** sizeLimit caps the emptyDir at 100Mi. Writing 110MB exceeds it, so Kubernetes evicts the pod. A controller would then recreate it.

**Proof:** `kubectl describe pod` showing SizeLimit 100Mi; `kubectl get pods` showing status Evicted after the write.

---

## Q31: generic Secret from literals; base64 not encrypted

```bash
kubectl create secret generic db-secret --from-literal=username=admin --from-literal=password=mypassword
kubectl get secret db-secret -o yaml
echo YWRtaW4= | base64 --decode
```

**Explanation:** Secrets are only base64-encoded, not encrypted. Anyone with read access can decode them instantly, as I show by decoding the username back to "admin". Real protection needs etcd encryption-at-rest or a secrets manager like Vault.

**Proof:** the -o yaml output showing base64 values; the decode producing "admin".

---

## Q32: Secret from files; identify keys

```bash
echo "fake-cert" > cert.pem
echo "fake-key" > key.pem
kubectl create secret generic tls-files --from-file=cert.pem=cert.pem --from-file=key.pem=key.pem
kubectl describe secret tls-files
```

**Explanation:** With --from-file the key name inside the secret equals the filename before the = sign, so the resulting keys are cert.pem and key.pem. (Generic secrets don't validate content, so dummy text is fine here.)

**Proof:** `kubectl describe secret tls-files` listing the two keys cert.pem and key.pem.

---

## Q33: kubernetes.io/tls Secret; where consumed

```bash
openssl req -x509 -nodes -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365 -subj "/CN=example.com"
kubectl create secret tls my-tls-secret --cert=cert.pem --key=key.pem
kubectl describe secret my-tls-secret
```

**Explanation:** The TLS secret type validates real PEM data, so I generated a proper self-signed cert and key with openssl first. A kubernetes.io/tls secret holds tls.crt and tls.key and is consumed by Ingress controllers or OpenShift Routes to terminate HTTPS on behalf of the app.

**Proof:** `kubectl describe secret my-tls-secret` showing Type kubernetes.io/tls with tls.crt and tls.key.

---

## Q34: Mount Secret as volume; trade-offs vs env vars

```bash
nano secret-volume-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-volume-pod
spec:
  volumes:
  - name: creds
    secret:
      secretName: db-secret
  containers:
  - name: app
    image: docker.io/library/busybox:latest
    command: ['sleep','3600']
    volumeMounts:
    - name: creds
      mountPath: /etc/secrets
```

```bash
kubectl apply -f secret-volume-pod.yaml
kubectl exec secret-volume-pod -- ls /etc/secrets
kubectl exec secret-volume-pod -- cat /etc/secrets/username
```

**Explanation:** As a volume, each secret key becomes a file. Advantages over env vars: good for apps that read config files, the files auto-update when the secret changes, and I can set restrictive permissions. Env vars are simpler to read in code but leak more easily in logs, don't auto-update, and show in describe pod.

**Proof:** `ls /etc/secrets` listing the keys; `cat .../username` showing admin.

---

## Q35: envFrom with secretRef loads all keys

```bash
nano envfrom-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envfrom-pod
spec:
  containers:
  - name: app
    image: docker.io/library/busybox:latest
    command: ['sleep','3600']
    envFrom:
    - secretRef:
        name: db-secret
```

```bash
kubectl apply -f envfrom-pod.yaml
kubectl exec envfrom-pod -- env | grep -E "username|password"
```

**Explanation:** envFrom with secretRef loads every key in the secret as an environment variable at once, so I don't have to list each key individually.

**Proof:** the env output showing username and password as environment variables.

---

## Q36: docker-registry Secret + imagePullSecrets; when required

```bash
kubectl create secret docker-registry my-registry-secret --docker-server=registry.example.com --docker-username=myuser --docker-password=mypassword
kubectl get secret my-registry-secret
nano pullsecret-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pullsecret-pod
spec:
  imagePullSecrets:
  - name: my-registry-secret
  containers:
  - name: app
    image: registry.example.com/myapp:latest
```

```bash
kubectl apply -f pullsecret-pod.yaml
```

**Explanation:** A docker-registry secret holds registry credentials, referenced via imagePullSecrets so Kubernetes can authenticate. It's required when pulling from a private registry or when hitting Docker Hub's anonymous rate limit; without it you get ImagePullBackOff.

**Proof:** `kubectl describe secret my-registry-secret` showing type dockerconfigjson; the pod referencing it.

---

## Q37: Secret with several keys, selectively mount one

```bash
kubectl create secret generic multi-secret --from-literal=db_user=admin --from-literal=db_pass=secret --from-literal=api_key=xyz123
nano selective-secret-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: selective-secret-pod
spec:
  volumes:
  - name: pass-only
    secret:
      secretName: multi-secret
      items:
      - key: db_pass
        path: password.txt
  containers:
  - name: app
    image: docker.io/library/busybox:latest
    command: ['sleep','3600']
    volumeMounts:
    - name: pass-only
      mountPath: /etc/secrets
```

```bash
kubectl apply -f selective-secret-pod.yaml
kubectl exec selective-secret-pod -- ls /etc/secrets
kubectl exec selective-secret-pod -- cat /etc/secrets/password.txt
```

**Explanation:** The items section selects specific keys and controls their filenames, so only db_pass is mounted (as password.txt) even though the secret has three keys.

**Proof:** `ls /etc/secrets` showing only password.txt; `cat` showing secret.

---

## Q38: ClusterIP Service + DNS from another pod

```bash
kubectl expose deployment web --name=web-svc --port=80
kubectl get service web-svc
kubectl run dns-test --rm -it --image=docker.io/library/busybox:latest -- sh
# inside: nslookup web-svc.default.svc.cluster.local ; wget -qO- http://web-svc
```

**Explanation:** ClusterIP gives the service a stable internal IP reachable only inside the cluster. DNS resolves it as service.namespace.svc.cluster.local, and within the same namespace the short name web-svc works.

**Proof:** nslookup resolving the name; wget returning the nginx page.

---

## Q39: NodePort Service and reach it

```bash
kubectl expose deployment web --type=NodePort --port=80
kubectl get service web
curl $(minikube ip):<nodeport>
```

**Explanation:** NodePort exposes the service on a high port (30000–32767) on every node IP. I reached it by curling the node IP on that port (more fundamental than the minikube service helper), which returned the nginx page.

**Proof:** `kubectl get service web` showing the NodePort; the curl output showing the nginx welcome HTML.

---

## Q40: LoadBalancer Service; why EXTERNAL-IP pending on minikube

```bash
kubectl expose deployment web --type=LoadBalancer --port=80
kubectl get service web        # EXTERNAL-IP <pending>
minikube tunnel                # in a separate terminal
kubectl get service web        # now has an IP
```

**Explanation:** EXTERNAL-IP stays pending because a LoadBalancer needs a cloud provider to provision a real external IP, and minikube has none. minikube tunnel simulates one locally.

**Proof:** the service showing <pending> first, then an external IP after tunnel.

---

## Q41: Headless Service for StatefulSet; per-pod A records

```bash
kubectl get service redis        # CLUSTER-IP None
kubectl get endpoints redis
kubectl run dns-test --restart=Never --image=docker.io/library/busybox:latest -- nslookup redis.default.svc.cluster.local
kubectl logs dns-test
kubectl delete pod dns-test
```

**Explanation:** A headless service (clusterIP: None) has no virtual IP; DNS instead returns the IP of every individual pod, so clients can target a specific pod like redis-0.redis.default.svc.cluster.local.

**Proof:** nslookup returning three A records (one per redis pod); endpoints listing the three IPs.

---

## Q42: Inspect Endpoints; how populated

```bash
kubectl get endpoints web-svc
kubectl get pods --show-labels
kubectl describe service web-svc
```

**Explanation:** Endpoints are filled automatically: Kubernetes watches for pods whose labels match the service selector, and a pod's IP is added once it's Running and passing its readiness probe — and removed when it fails or is deleted.

**Proof:** endpoints listing the pod IPs; the matching labels/selector.

---

## Q43: Cross-namespace access via FQDN; short name fails

```bash
kubectl create namespace test-ns
kubectl run test-pod --rm -it --image=docker.io/library/busybox:latest -n test-ns -- sh
# inside: wget web-svc            (fails)
# inside: wget web-svc.default.svc.cluster.local   (works)
```

**Explanation:** Short service names only resolve within the same namespace, so from test-ns the short name fails; the FQDN service.namespace.svc.cluster.local works from anywhere.

**Proof:** the short-name wget failing to resolve, the FQDN wget succeeding.

---

## Q44: port-forward to Service vs Pod

```bash
kubectl port-forward pod/<web-pod> 8080:80      # one specific pod
kubectl port-forward service/web-svc 8080:80    # load-balances across pods
```

**Explanation:** Forwarding to a pod targets that one pod and breaks if it dies — good for debugging one pod. Forwarding to a service goes through the service layer and load-balances across all matching pods, so it survives a pod dying — good for testing the app end-to-end.

**Proof:** curl localhost:8080 returning the app in each case.

---

## Q45: port vs targetPort vs nodePort

```bash
kubectl describe service web-svc        # shows Port, TargetPort, NodePort
```

**Explanation:** port is what clients use to reach the Service; targetPort is the container port the app listens on; nodePort is the port on each node IP for outside access (NodePort services only). Flow: client → Service port → Pod targetPort.

**Proof:** `kubectl describe service` showing the three fields.

---

## Q46: Verify ClusterIP connectivity from throwaway pod

```bash
kubectl run tmp --rm -it --image=docker.io/library/busybox:latest -- sh
# inside: wget -qO- http://web-svc ; nc -z web-svc 80 && echo connected
```

**Explanation:** A throwaway pod with --rm deletes itself on exit, giving a clean way to test internal connectivity without leaving pods behind.

**Proof:** wget returning the page and/or "connected" from the nc check.

---

## Q47: Two Deployments, one Service load-balancing via shared label

```bash
kubectl create deployment app-v1 --image=docker.io/library/nginx:1.25
kubectl create deployment app-v2 --image=docker.io/library/httpd:2.4
# give both deployments' pods the shared label app=myapp (edit each deployment's template labels)
kubectl create service clusterip myapp-svc --tcp=80:80
# ensure the service selector is app: myapp
kubectl get endpoints myapp-svc
```

**Explanation:** When pods from two deployments share the label app=myapp, a single Service selecting that label load-balances across pods from both. The proof is endpoints containing IPs from both deployments. (Set the label in the deployment template, not on live pods, or the ReplicaSet reverts it; and make sure the service selector is app: myapp.)

**Proof:** `kubectl get endpoints myapp-svc` listing pods from both app-v1 and app-v2.

---

## Q48: Diagnose why a pod can't reach a Service

```bash
kubectl exec <pod> -- nslookup web-svc        # 1 DNS
kubectl get endpoints web-svc                 # 2 endpoints present?
kubectl describe service web-svc              # 3 selector
kubectl get pods --show-labels                # 3 labels match?
kubectl describe pod <pod>                     # 4 readiness
kubectl describe service web-svc              # 5 ports
```

**Explanation:** I check in order — DNS resolves the name, endpoints exist (else no pods match the selector), pod labels match the service selector, pods pass readiness, and the port/targetPort are correct. Each step isolates a different failure cause.

**Proof:** the outputs at whichever step reveals the problem (e.g. empty endpoints, label mismatch).

---

## Q49: NodePort and verify it works

```bash
kubectl expose deployment web --type=NodePort --port=80 --name=web-nodeport
NODE_PORT=$(kubectl get svc web-nodeport -o jsonpath='{.spec.ports[0].nodePort}')
curl http://$(minikube ip):$NODE_PORT
```

**Explanation:** I exposed the deployment as NodePort, read the assigned port programmatically, and curled the node IP on that port to confirm it serves traffic.

**Proof:** the curl returning the nginx page.

---

## Q50: HTTP livenessProbe on / port 80 (nginx)

```bash
kubectl patch deployment web --patch '{"spec":{"template":{"spec":{"containers":[{"name":"nginx","livenessProbe":{"httpGet":{"path":"/","port":80},"initialDelaySeconds":5,"periodSeconds":10}}]}}}}'
kubectl describe $(kubectl get pods -l app=web -o name | head -1)
```

**Explanation:** A livenessProbe checks the app is still alive; if it fails repeatedly Kubernetes restarts the container. initialDelaySeconds delays the first check; periodSeconds sets the interval.

**Proof:** `kubectl describe pod` showing the Liveness http-get / on port 80.

---

## Q51: tcpSocket readinessProbe on redis 6379

```bash
kubectl patch statefulset redis --patch '{"spec":{"template":{"spec":{"containers":[{"name":"redis","readinessProbe":{"tcpSocket":{"port":6379},"initialDelaySeconds":5,"periodSeconds":5}}]}}}}'
kubectl describe pod redis-0
```

**Explanation:** A readinessProbe decides if a pod gets traffic; if it fails the pod is removed from Service endpoints. A tcpSocket probe just opens a connection on the port — no HTTP needed.

**Proof:** `kubectl describe pod redis-0` showing the Readiness tcp-socket on 6379.

---

## Q52: startupProbe for ~2-minute boot

```bash
nano startup-probe-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: startup-probe-pod
spec:
  containers:
  - name: app
    image: docker.io/library/busybox:latest
    command: ['sleep','3600']
    startupProbe:
      httpGet:
        path: /health
        port: 8080
      failureThreshold: 24
      periodSeconds: 5
```

```bash
kubectl apply -f startup-probe-pod.yaml
kubectl describe pod startup-probe-pod
```

**Explanation:** A startupProbe gives slow apps time to boot before liveness takes over. failureThreshold × periodSeconds must be ≥ 120 for two minutes, so 24 × 5 = 120 seconds. Without it, liveness might kill an app that's still starting.

**Proof:** `kubectl describe pod` showing the Startup probe with threshold 24 and period 5s.

---

## Q53: Default behavior with no probes (theory)

**Explanation:** With no probes, Kubernetes assumes a container is ready and healthy as soon as it starts and never checks the app. If the process crashes it restarts it (it sees the exit), but if the app is running yet broken — stuck, deadlocked, not serving — Kubernetes can't tell and keeps sending traffic to it. That's why probes matter: startupProbe knows when a slow app finished starting, readinessProbe knows when it's ready for traffic, and livenessProbe knows when it's broken and needs a restart.

**Proof:** none needed — this is a theory answer.
