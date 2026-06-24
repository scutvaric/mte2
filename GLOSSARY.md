# Intro to DevOps — Exam Glossary

Open-book reference for the practice questions (LO4, LO5, LO6).
Organized **by Learning Outcome**, then **by question number**. Each Learning Outcome
starts with a clickable index: find your question number, click, jump to the answer.

Practical entries (LO4, LO5) are written as a **mini-runbook**: the exact command(s) to run
(often **Method A = one command**, **Method B = edit a file**), followed by **Verify** —
how to confirm it worked. Theory (LO6) is **ready-to-write prose**: first sentence is your
*standing*, then the arguments.

> **New here? Read [CS-0 (editing & applying workflow + surviving vi)](#lo4-cs0) once — it explains how to actually run the edits below.**

---

## Table of Contents

- [LO4 — Kubernetes objects & delivery](#lo4--accelerated-delivery-of-multilayer-applications-using-containers)
  - [CS-0 editing & applying workflow (vi survival)](#lo4-cs0)
  - [Cheat-sheets (quick commands, Service types, probes)](#lo4-cheatsheets)
  - [Question index](#lo4-index)
- [LO5 — Troubleshooting](#lo5--solve-problems-with-application-shipping-by-using-containers)
  - [Cheat-sheets (diagnostic trio, pod statuses)](#lo5-cheatsheets)
  - [Question index](#lo5-index)
- [LO6 — Orchestration systems (theory)](#lo6--evaluate-the-use-of-selected-container-orchestration-systems-theory)
  - [Cheat-sheets (shared comparisons)](#lo6-cheatsheets)
  - [Question index](#lo6-index)

---

<a id="lo4--accelerated-delivery-of-multilayer-applications-using-containers"></a>
# LO4 — Accelerated delivery of multilayer applications using containers

Practical Kubernetes. Each entry gives the **full question**, then the way(s) to do it, then
**Verify**. Where it helps you see both styles, **Method A** is a one-shot command and
**Method B** edits YAML. Read CS-0 once so the edit/save steps make sense.

<a id="lo4-cs0"></a>
## CS-0 · Editing & applying workflow — READ ONCE

**Three ways to change a Kubernetes object:**
- **Imperative command** — fastest, no YAML to write. Examples: `kubectl set image`, `kubectl set resources`, `kubectl scale`, `kubectl expose`, `kubectl label`, `kubectl patch`. Use one if it exists.
- **`kubectl edit <kind>/<name>`** — opens the *live* object in an editor; you change it, save, and Kubernetes applies it instantly. Good for one-off tweaks.
- **Edit a file, then `kubectl apply -f file.yaml`** — best when you want a saved manifest you can re-apply or push to Git.

**Surviving `vi` (the default editor for `kubectl edit`):**
- It opens in **normal mode** — keys are *commands*, not text. Don't type yet.
- Move to the line with the **arrow keys**.
- Press **`i`** to start typing (**insert mode**). Indent with the **spacebar** — there's no auto-indent, you type the spaces yourself.
- Press **`Esc`** to leave insert mode.
- **Save & quit:** `Esc`, then type `:wq`, then `Enter`.
- **Quit WITHOUT saving** (start over): `Esc`, then type `:q!`, then `Enter`.
- Hate `vi`? Run `export KUBE_EDITOR=nano` once before `kubectl edit` — nano shows `^O` (save) and `^X` (exit) at the bottom and needs no mode-switching.

**Verifying any change** (works for everything below):
```bash
kubectl get <kind>/<name> -o yaml          # full object
kubectl get <kind>/<name> -o jsonpath='{...}'   # one field
kubectl describe <kind>/<name>             # human-readable + Events
```

**Tip — generate correct YAML instead of writing it by hand:**
```bash
kubectl create deployment x --image=nginx --dry-run=client -o yaml > x.yaml
```
`--dry-run=client -o yaml` prints a valid manifest **without creating anything**, so you edit a correct skeleton instead of typing from memory.

<a id="lo4-cheatsheets"></a>
## Cheat-sheets

### CS-1 · Imperative quick commands (fast points)
```
kubectl create deployment web --image=nginx:1.25 --replicas=3
kubectl scale deployment/web --replicas=5
kubectl set image deployment/web nginx=nginx:1.27        # rolling update
kubectl set resources deployment/web --requests=cpu=100m,memory=128Mi --limits=cpu=500m,memory=256Mi
kubectl rollout status deployment/web
kubectl rollout history deployment/web
kubectl rollout undo deployment/web                      # rollback
kubectl expose deployment web --port=80 --type=ClusterIP
kubectl get deploy,rs,pod -o wide
kubectl run tmp --rm -it --image=busybox -- sh           # throwaway debug pod
```
Add `--dry-run=client -o yaml > file.yaml` to any `create`/`run` to generate a manifest without applying it.

### CS-2 · Service types (port / targetPort / nodePort)
- **ClusterIP** (default): virtual IP reachable only inside the cluster; DNS `svc.ns.svc.cluster.local`.
- **NodePort**: ClusterIP + a port (30000–32767) on every node → reachable via `nodeIP:nodePort`.
- **LoadBalancer**: NodePort + a cloud LB. On minikube `EXTERNAL-IP` stays `<pending>` → `minikube tunnel`.
- **Headless** (`clusterIP: None`): no VIP; DNS returns **one A record per pod**. Used by StatefulSets.
- **Three ports:** `port` = Service's own port; `targetPort` = container port forwarded to; `nodePort` = external port on the node.

### CS-3 · Probes
- **liveness** → restart container on failure. **readiness** → drop pod from Service endpoints on failure (no restart). **startup** → gate the other two during a slow boot.
- Handlers: `httpGet`, `tcpSocket`, `exec`. Startup budget = `failureThreshold × periodSeconds`.
- **No probes** → Ready as soon as the process starts; restarted only if the process exits.

---

<a id="lo4-index"></a>
## LO4 — Question index

**Deployments / rollouts:** [1](#lo4-1) [3](#lo4-3) [4](#lo4-4) [5](#lo4-5) [6](#lo4-6) [7](#lo4-7) [8](#lo4-8) [9](#lo4-9) [10](#lo4-10) [11](#lo4-11) [12](#lo4-12) [13](#lo4-13) [14](#lo4-14)
**Pods / workload controllers:** [15](#lo4-15) [16](#lo4-16) [17](#lo4-17) [18](#lo4-18) [19](#lo4-19) [20](#lo4-20) [21](#lo4-21) [22](#lo4-22)
**Storage / volumes:** [23](#lo4-23) [24](#lo4-24) [25](#lo4-25) [27](#lo4-27) [28](#lo4-28) [29](#lo4-29) [30](#lo4-30)
**ConfigMaps / Secrets:** [2](#lo4-2) [26](#lo4-26) [31](#lo4-31) [32](#lo4-32) [33](#lo4-33) [34](#lo4-34) [35](#lo4-35) [36](#lo4-36) [37](#lo4-37)
**Services / networking:** [38](#lo4-38) [39](#lo4-39) [40](#lo4-40) [41](#lo4-41) [42](#lo4-42) [43](#lo4-43) [44](#lo4-44) [45](#lo4-45) [46](#lo4-46) [47](#lo4-47) [48](#lo4-48) [49](#lo4-49)
**Probes:** [50](#lo4-50) [51](#lo4-51) [52](#lo4-52) [53](#lo4-53)

---

<a id="lo4-1"></a>
### Q1 · Deployment web, nginx:1.25, 3 replicas (imperative) + export YAML

> **Full question:** Create a Deployment named web running nginx:1.25 with 3 replicas using a single imperative kubectl create deployment command, then export its generated YAML to a file.

```bash
kubectl create deployment web --image=nginx:1.25 --replicas=3
kubectl get deployment web -o yaml > web.yaml
```
**Verify:** `kubectl get deploy web` shows `READY 3/3`; `ls web.yaml` exists.
*Note:* one command creates it; `-o yaml > file` exports the generated manifest.

<a id="lo4-2"></a>
### Q2 · Pull from Docker Hub authenticated to avoid the pull limit — what resource?

> **Full question:** Enable pulling images from DockerHub as an authenticated user to lower the chance of hitting the pull limit. What kind of resource do you need to create?

**Answer: a docker-registry (image-pull) Secret**, referenced via `imagePullSecrets`.
```bash
kubectl create secret docker-registry dhcred \
  --docker-server=docker.io --docker-username=USER \
  --docker-password=PASS --docker-email=you@example.com
```
Then add to the pod/deployment spec: `spec.imagePullSecrets: [{ name: dhcred }]`.
**Verify:** `kubectl get secret dhcred` (type `kubernetes.io/dockerconfigjson`).
*Why:* authenticated pulls get a higher Docker Hub rate limit than anonymous. (Same as Q36.)

<a id="lo4-3"></a>
### Q3 · Scale 3 → 5 two ways + show the ReplicaSet

> **Full question:** Scale web from 3 to 5 replicas two different ways — once with kubectl scale, once by editing the manifest — and show the ReplicaSet that results.

**Method A — command:**
```bash
kubectl scale deployment/web --replicas=5
```
**Method B — edit the live object:** `kubectl edit deployment/web`, change `spec.replicas:` to `5`, save & quit (CS-0).
**Verify:**
```bash
kubectl get deploy web          # READY 5/5
kubectl get rs -l app=web       # the ReplicaSet shows DESIRED 5
```
*Why no new RS:* only `spec.replicas` changed, so the existing ReplicaSet scales — a *new* RS appears only on a pod-template change (e.g. image update).

<a id="lo4-4"></a>
### Q4 · Rolling update nginx:1.25 → 1.27 + watch

> **Full question:** Perform a rolling update of web from nginx:1.25 to nginx:1.27 and watch it with kubectl rollout status.

**Method A — command (recommended):**
```bash
kubectl set image deployment/web nginx=nginx:1.27
kubectl rollout status deployment/web
```
**Method B — edit:** `kubectl edit deployment/web`, change `image: nginx:1.25` to `nginx:1.27`, save & quit, then `kubectl rollout status deployment/web`.
**Verify:** `rollout status` prints "successfully rolled out"; `kubectl get pods` shows new pods.
*Why:* changing the image updates the pod template → a **new ReplicaSet** is created and pods are replaced gradually (default RollingUpdate). The container name here is `nginx` (derived from the image at create time).

<a id="lo4-5"></a>
### Q5 · Rollout history, rollback, CHANGE-CAUSE

> **Full question:** View a Deployment's rollout history and roll back to the previous revision. Explain what the CHANGE-CAUSE column shows and how to populate it.

```bash
kubectl rollout history deployment/web
kubectl rollout undo deployment/web                 # back to previous revision
kubectl rollout undo deployment/web --to-revision=2 # a specific one
```
**Populate CHANGE-CAUSE:** `kubectl annotate deployment/web kubernetes.io/change-cause="upgrade to 1.27"` (or use `--record` on the changing command).
**Verify:** `kubectl rollout history deployment/web` shows the revisions and your change-cause text.
*Why:* CHANGE-CAUSE is just an annotation describing each revision; it's empty unless you set it.

<a id="lo4-6"></a>
### Q6 · RollingUpdate maxSurge: 1, maxUnavailable: 0

> **Full question:** Set the update strategy to RollingUpdate with maxSurge: 1 and maxUnavailable: 0, and explain the effect on availability during an update.

**Method A — patch (one command):**
```bash
kubectl patch deployment web -p \
'{"spec":{"strategy":{"type":"RollingUpdate","rollingUpdate":{"maxSurge":1,"maxUnavailable":0}}}}'
```
**Method B — edit:** `kubectl edit deployment/web`, set under `spec.strategy` (save & quit, CS-0):
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
```
**Verify:** `kubectl get deployment web -o jsonpath='{.spec.strategy}'; echo`
*Effect:* `maxUnavailable: 0` keeps full capacity (no old pod removed until a new one is Ready); `maxSurge: 1` allows one extra pod → zero-downtime update at the cost of briefly running replicas+1.

<a id="lo4-7"></a>
### Q7 · Recreate strategy + when it's required

> **Full question:** Switch a Deployment's strategy to Recreate and describe a concrete scenario where this is required instead of RollingUpdate.

**Method A — patch:**
```bash
kubectl patch deployment web -p '{"spec":{"strategy":{"type":"Recreate","rollingUpdate":null}}}'
```
**Method B — edit:** set `spec.strategy.type: Recreate` (remove the `rollingUpdate` block), save & quit.
**Verify:** `kubectl get deployment web -o jsonpath='{.spec.strategy.type}'; echo` → `Recreate`.
*Why/when:* Recreate kills all old pods first, then starts new ones (brief downtime). Required when old and new can't coexist — e.g. a single-writer DB schema migration, or a `ReadWriteOnce` volume only one pod can mount.

<a id="lo4-8"></a>
### Q8 · CPU/memory requests + limits, verify in the running pod

> **Full question:** Add CPU/memory requests and limits to a Deployment's container and verify they appear in the running pod spec.

**Method A — command (recommended):**
```bash
kubectl set resources deployment web \
  --requests=cpu=100m,memory=128Mi \
  --limits=cpu=500m,memory=256Mi
```
**Method B — edit:** `kubectl edit deployment/web`; under the container (aligned with its `name:`) replace `resources: {}` with — then save & quit (CS-0):
```yaml
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "256Mi"
```
**Verify:**
```bash
kubectl get pod -l app=web -o jsonpath='{.items[0].spec.containers[0].resources}'; echo
```
*Why:* `requests` affect scheduling (node must have them free); `limits` cap usage (over-limit memory → OOMKilled, LO5 #23).

<a id="lo4-9"></a>
### Q9 · revisionHistoryLimit: 3

> **Full question:** Set revisionHistoryLimit: 3 and explain how it affects your ability to roll back and how many old ReplicaSets are kept.

**Method A — patch:** `kubectl patch deployment web -p '{"spec":{"revisionHistoryLimit":3}}'`
**Method B — edit:** set `spec.revisionHistoryLimit: 3`, save & quit.
**Verify:** `kubectl get deployment web -o jsonpath='{.spec.revisionHistoryLimit}'; echo` → `3`; `kubectl get rs -l app=web` keeps at most 3 old (0-replica) ReplicaSets.
*Why:* it caps how many old ReplicaSets (rollback targets) are retained; older ones are garbage-collected and can no longer be rolled back to.

<a id="lo4-10"></a>
### Q10 · Bad image tag → rollout stuck, old pods keep serving

> **Full question:** Set a Deployment's image to a non-existent tag, observe the rollout get stuck, and explain why the old pods keep serving traffic.

```bash
kubectl set image deployment/web nginx=nginx:doesnotexist
kubectl rollout status deployment/web        # hangs
kubectl get pods                             # new pod in ImagePullBackOff
```
**Recover:** `kubectl rollout undo deployment/web` (or set a valid tag).
*Why:* with `maxUnavailable: 0` (or default), Kubernetes won't delete old pods until new ones are Ready — and they never become Ready, so the **old ReplicaSet keeps serving**. The rollout pauses; it's not destructive.

<a id="lo4-11"></a>
### Q11 · Label selector to list one Deployment's pods + the selector chain

> **Full question:** Use a label selector to list only the pods belonging to one Deployment, and explain the Deployment → ReplicaSet → Pod selector relationship.

```bash
kubectl get pods -l app=web
```
**Verify:** only that Deployment's pods are listed.
*Why:* a **Deployment** owns a **ReplicaSet** via `spec.selector.matchLabels`, and the RS owns **Pods** whose `template.metadata.labels` match. Labels must align across all three or the controller won't recognize its pods (LO5 #26).

<a id="lo4-12"></a>
### Q12 · kubectl expose — what object, how is the selector derived?

> **Full question:** Expose a Deployment with kubectl expose, then explain what object was created and how its selector was derived.

```bash
kubectl expose deployment web --port=80 --target-port=80
```
**Verify:** `kubectl get svc web`; `kubectl get svc web -o jsonpath='{.spec.selector}'; echo`
*Why:* creates a **Service** (ClusterIP by default); its `selector` is **copied from the Deployment's pod-template labels**, so it automatically targets that Deployment's pods.

<a id="lo4-13"></a>
### Q13 · Sidecar (second container) + how it shares the pod

> **Full question:** Add a sidecar (second) container to a Deployment's pod template and explain how the two containers share the pod network and volumes.

**Method B — edit:** `kubectl edit deployment/web`; under `spec.template.spec.containers` add a second list item (same indentation as the first `- name:`), then save & quit:
```yaml
      - name: sidecar
        image: busybox
        command: ["sh","-c","sleep infinity"]
```
**Verify:** `kubectl get pod -l app=web -o jsonpath='{.items[0].spec.containers[*].name}'; echo` shows both names.
*Why:* both containers share the pod's **network namespace** (same IP, reach each other on `localhost`) and any **volume mounted into both**; they don't share filesystems otherwise.

<a id="lo4-14"></a>
### Q14 · Deployment httpd:2.4, 2 replicas, named port, nodeSelector → why Pending

> **Full question:** Write a Deployment manifest for httpd:2.4 with 2 replicas, a named container port, and a nodeSelector; explain why it stays Pending if no node matches.

Write `httpd.yaml` (generate a skeleton with `kubectl create deployment httpd --image=httpd:2.4 --replicas=2 --dry-run=client -o yaml > httpd.yaml`, then add the port + nodeSelector), then `kubectl apply -f httpd.yaml`:
```yaml
    spec:
      nodeSelector: { disktype: ssd }
      containers:
      - name: httpd
        image: httpd:2.4
        ports:
        - name: http
          containerPort: 80
```
**Verify (the Pending case):** `kubectl get pods` → Pending; `kubectl describe pod <p>` → "0/N nodes match node selector".
**Fix:** `kubectl label node <node> disktype=ssd` (or remove the selector).
*Why:* if no node has `disktype=ssd`, the scheduler can't place the pods.

<a id="lo4-15"></a>
### Q15 · Bare Pod (no controller) busybox sleep — delete vs Deployment-managed

> **Full question:** Create a bare Pod (no controller) running busybox that sleeps, then explain what happens when you delete it versus a Deployment-managed pod.

```bash
kubectl run c1 --image=busybox --restart=Never -- sleep 3600
kubectl delete pod c1            # gone — nothing recreates it
```
**Verify:** after delete, `kubectl get pod c1` → NotFound. A Deployment's pod, by contrast, is recreated by its ReplicaSet.
*Why:* a bare Pod has no controller, so deletion is final; use bare pods only for one-off debugging.

<a id="lo4-16"></a>
### Q16 · StatefulSet redis:7, 3 replicas + headless Service → stable names & DNS

> **Full question:** Write a StatefulSet for redis:7 with 3 replicas plus a headless Service, and show the stable pod names and per-pod DNS records.

Write `redis.yaml` and `kubectl apply -f redis.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata: { name: redis }
spec:
  clusterIP: None            # headless
  selector: { app: redis }
  ports: [{ port: 6379 }]
---
apiVersion: apps/v1
kind: StatefulSet
metadata: { name: redis }
spec:
  serviceName: redis
  replicas: 3
  selector: { matchLabels: { app: redis } }
  template:
    metadata: { labels: { app: redis } }
    spec:
      containers:
      - { name: redis, image: redis:7, ports: [{ containerPort: 6379 }] }
```
**Verify:**
```bash
kubectl get pods -l app=redis          # redis-0, redis-1, redis-2 (ordered)
kubectl run tmp --rm -it --image=busybox -- nslookup redis-0.redis.default.svc.cluster.local
```
*Why:* StatefulSet gives **stable ordinal names** and per-pod DNS `redis-0.redis.<ns>.svc.cluster.local` — stable identity + per-pod storage is what separates it from a Deployment.

<a id="lo4-17"></a>
### Q17 · DaemonSet busybox — why exactly one pod per node

> **Full question:** Create a DaemonSet running a busybox agent and explain why exactly one pod runs per node.

Write `ds.yaml` with `kind: DaemonSet` (pod template runs e.g. `busybox sleep infinity`), then `kubectl apply -f ds.yaml`.
**Verify:** `kubectl get ds` → DESIRED == number of nodes; `kubectl get pods -o wide` → one per node.
*Why:* a DaemonSet's controller schedules **one pod on every matching node** and adds one when a node joins — by design, not replica-count driven. Used for node-level agents (logging, monitoring, CNI).

<a id="lo4-18"></a>
### Q18 · Job busybox/perl that runs once + read result

> **Full question:** Create a Job using busybox/perl that computes something once and completes; show COMPLETIONS and read the result from logs.

```bash
kubectl create job pi --image=perl -- perl -Mbignum=bpi -wle 'print bpi(20)'
```
**Verify:**
```bash
kubectl get job pi        # COMPLETIONS 1/1
kubectl logs job/pi       # the computed value
```
*Why:* a **Job** runs a pod to successful completion, then stops; output lives in the pod's logs.

<a id="lo4-19"></a>
### Q19 · CronJob every minute + suspend + list spawned Jobs

> **Full question:** Create a CronJob that prints the date every minute; show how to suspend it and how to list the Jobs it spawns.

```bash
kubectl create cronjob date --image=busybox --schedule="* * * * *" -- date
kubectl patch cronjob date -p '{"spec":{"suspend":true}}'   # pause it
kubectl get jobs                                            # Jobs it spawned
```
**Verify:** `kubectl get cronjob date` shows `SUSPEND True`; `kubectl get jobs` lists `date-<timestamp>` Jobs.
*Why:* a CronJob creates a **Job** each schedule tick; `suspend: true` stops new ones without deleting the CronJob.

<a id="lo4-20"></a>
### Q20 · Manually run a Job from a CronJob (on-demand test)

> **Full question:** Manually run a Job from an existing CronJob (kubectl create job --from=cronjob/...) to test it on demand.

```bash
kubectl create job manual-1 --from=cronjob/date
```
**Verify:** `kubectl get job manual-1` → runs immediately; `kubectl logs job/manual-1`.
*Why:* `--from=cronjob/...` clones the CronJob's job template into a one-off Job now, without waiting for the schedule.

<a id="lo4-21"></a>
### Q21 · StatefulSet ordered create/terminate vs Deployment

> **Full question:** Show that a StatefulSet creates and terminates pods in order, and contrast this with a Deployment.

```bash
kubectl get pods -l app=redis -w        # watch: redis-0 Ready, then -1, then -2
kubectl scale statefulset redis --replicas=0   # watch reverse order: -2, -1, -0
```
**Verify:** the watch shows creation in order and deletion in reverse.
*Why:* a StatefulSet brings pods up in order (waiting for each to be Ready) and tears them down in reverse; a Deployment creates/deletes in parallel with no ordering or stable names. Ordering matters for clustered stateful systems.

<a id="lo4-22"></a>
### Q22 · Multi-container Pod sharing an emptyDir

> **Full question:** Create a multi-container Pod where two containers share an emptyDir — one writes a file, the other reads it. Prove it's shared.

Write `shared.yaml` and `kubectl apply -f shared.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata: { name: shared }
spec:
  volumes:
  - { name: data, emptyDir: {} }
  containers:
  - name: writer
    image: busybox
    command: ["sh","-c","echo hello > /data/msg; sleep infinity"]
    volumeMounts: [{ name: data, mountPath: /data }]
  - name: reader
    image: busybox
    command: ["sh","-c","sleep infinity"]
    volumeMounts: [{ name: data, mountPath: /data }]
```
**Verify:** `kubectl exec shared -c reader -- cat /data/msg` → `hello`.
*Why:* an `emptyDir` lives for the pod's lifetime and is shared by every container that mounts it.

<a id="lo4-23"></a>
### Q23 · List StorageClasses, PVs, PVCs

> **Full question:** List the StorageClasses, PVs and PVCs.

```bash
kubectl get storageclass        # or: sc
kubectl get pv
kubectl get pvc -A
```
*Why:* **StorageClass** = how volumes are provisioned; **PV** = an actual volume; **PVC** = a pod's claim that binds to a PV.

<a id="lo4-24"></a>
### Q24 · Mount a ConfigMap as a volume (each key → a file)

> **Full question:** Mount a ConfigMap as a volume so each key becomes a file, and verify the contents inside the pod.

Create the ConfigMap, then mount it (in the pod/deployment spec):
```bash
kubectl create configmap my-config --from-literal=greeting=hello --from-literal=mode=prod
```
```yaml
      volumes:
      - name: cfg
        configMap: { name: my-config }
      containers:
      - name: app
        image: nginx
        volumeMounts:
        - { name: cfg, mountPath: /etc/cfg }
```
**Verify:** `kubectl exec <pod> -- ls /etc/cfg` (a file per key) and `kubectl exec <pod> -- cat /etc/cfg/greeting`.
*Why:* each key becomes a file whose content is the value. Volume-mounted ConfigMaps update in-pod when changed (with delay) — unlike env vars.

<a id="lo4-25"></a>
### Q25 · Mount a single ConfigMap key with subPath + the update caveat

> **Full question:** Mount a single ConfigMap key to a specific path using subPath; explain when it's needed and its update caveat.

```yaml
        volumeMounts:
        - name: cfg
          mountPath: /etc/app/app.conf
          subPath: app.conf
```
**Verify:** `kubectl exec <pod> -- ls /etc/app` shows `app.conf` alongside any pre-existing files (the directory isn't hidden).
*Why/caveat:* `subPath` mounts **one key to a specific file** without masking the rest of the directory — but **subPath files do NOT auto-update** when the ConfigMap changes; you must restart the pod.

<a id="lo4-26"></a>
### Q26 · Mount a Secret as a volume (decoded values, restrictive perms)

> **Full question:** Mount a Secret as a volume and verify the files contain decoded values with restrictive permissions.

```yaml
      volumes:
      - name: sec
        secret: { secretName: my-secret, defaultMode: 0400 }
      containers:
      - name: app
        image: nginx
        volumeMounts: [{ name: sec, mountPath: /etc/sec, readOnly: true }]
```
**Verify:** `kubectl exec <pod> -- cat /etc/sec/<key>` shows the **decoded** value; `kubectl exec <pod> -- ls -l /etc/sec` shows `-r--------`.
*Why:* mounted Secret files are plaintext (not base64) with the mode you set.

<a id="lo4-27"></a>
### Q27 · Scaling a StatefulSet → one PVC per replica; what happens on delete

> **Full question:** Show that scaling a StatefulSet creates one PVC per replica, then explain what happens to those PVCs when the StatefulSet is deleted.

```bash
kubectl get pvc -l app=redis          # one PVC per replica (data-redis-0, -1, -2)
kubectl scale statefulset redis --replicas=4
kubectl get pvc -l app=redis          # a new PVC appears
kubectl delete statefulset redis
kubectl get pvc -l app=redis          # PVCs still there!
```
*Why:* each replica gets its own PVC from `volumeClaimTemplates`. **Deleting the StatefulSet does NOT delete the PVCs** (nor their data) by default — protects data, but can leave orphaned volumes you must remove manually.

<a id="lo4-28"></a>
### Q28 · initContainer pre-populates a shared volume

> **Full question:** Use an initContainer to pre-populate data into a shared volume before the main container reads it.

```yaml
    spec:
      volumes: [{ name: shared, emptyDir: {} }]
      initContainers:
      - name: seed
        image: busybox
        command: ["sh","-c","echo hello > /work/data"]
        volumeMounts: [{ name: shared, mountPath: /work }]
      containers:
      - name: app
        image: busybox
        command: ["sh","-c","cat /work/data; sleep infinity"]
        volumeMounts: [{ name: shared, mountPath: /work }]
```
**Verify:** `kubectl logs <pod>` shows `hello` (main container read what init wrote).
*Why:* **initContainers run to completion before the main container starts**, so the data is guaranteed present.

<a id="lo4-29"></a>
### Q29 · readOnly: true volume mount → writes rejected

> **Full question:** Set readOnly: true on a volume mount and prove writes are rejected.

```yaml
        volumeMounts:
        - { name: data, mountPath: /data, readOnly: true }
```
**Verify:** `kubectl exec <pod> -- touch /data/x` → "Read-only file system".
*Why:* used for config/reference data the app must not modify.

<a id="lo4-30"></a>
### Q30 · sizeLimit on emptyDir

> **Full question:** Set a sizeLimit on an emptyDir and explain what happens if the container exceeds it.

```yaml
      volumes:
      - name: scratch
        emptyDir: { sizeLimit: 100Mi }
```
**Verify:** write beyond 100Mi (`dd if=/dev/zero of=/scratch/big bs=1M count=200`) → the pod is **evicted** (`kubectl get pod` → Evicted).
*Why:* the kubelet evicts the pod when the emptyDir exceeds its limit — protects the node from a runaway container filling local disk.

<a id="lo4-31"></a>
### Q31 · Generic Secret from literals; show it's base64, not encrypted

> **Full question:** Create a generic Secret from literals for a DB username/password, then show the stored values are base64-encoded, not encrypted.

```bash
kubectl create secret generic dbcred \
  --from-literal=username=admin --from-literal=password=s3cr3t
```
**Verify:**
```bash
kubectl get secret dbcred -o jsonpath='{.data.password}'; echo     # base64 string
kubectl get secret dbcred -o jsonpath='{.data.password}' | base64 -d; echo   # → s3cr3t
```
*Why:* values are only **base64-encoded** (trivially reversible), **not encrypted** — confidentiality relies on RBAC and (optionally) etcd encryption-at-rest.

<a id="lo4-32"></a>
### Q32 · Secret from files (--from-file): resulting keys

> **Full question:** Create a Secret from files (--from-file) holding a certificate and key, and identify the resulting keys.

```bash
kubectl create secret generic tlsfiles --from-file=tls.crt --from-file=tls.key
```
**Verify:** `kubectl get secret tlsfiles -o jsonpath='{.data}'; echo` → keys `tls.crt`, `tls.key`.
*Why:* the **filenames become the keys**, contents become values. Rename with `--from-file=customkey=path`.

<a id="lo4-33"></a>
### Q33 · kubernetes.io/tls Secret from cert/key + where consumed

> **Full question:** Create a kubernetes.io/tls typed Secret from a cert/key pair and explain where it's consumed.

```bash
kubectl create secret tls web-tls --cert=tls.crt --key=tls.key
```
**Verify:** `kubectl get secret web-tls` → type `kubernetes.io/tls`; fixed keys `tls.crt`, `tls.key`.
*Why:* consumed by an **Ingress** (TLS termination) or any pod that mounts it for HTTPS.

<a id="lo4-34"></a>
### Q34 · Mount Secret as volume vs env vars — trade-offs

> **Full question:** Mount a Secret as a volume and explain the security trade-offs versus env vars.

**Volume:** files can have restrictive perms, support **auto-rotation** (updated in-pod when the Secret changes), and are less likely to leak via logs/`/proc`/crash dumps.
**Env vars:** simpler, but visible to child processes, easily leaked into logs or `kubectl describe`, and don't update without a restart.
*Standing:* prefer **volumes** for sensitive, rotating credentials.

<a id="lo4-35"></a>
### Q35 · envFrom + secretRef (load every key as env vars)

> **Full question:** Use envFrom with a secretRef to load every key of a Secret as env vars at once.

```yaml
        envFrom:
        - secretRef: { name: dbcred }
```
**Verify:** `kubectl exec <pod> -- env | grep -E 'username|password'`.
*Why:* loads **all keys** of the Secret as env vars at once (key → var name). Same idea with `configMapRef` for ConfigMaps; inherits the env-var leakage caveats of Q34.

<a id="lo4-36"></a>
### Q36 · docker-registry (image-pull) Secret + imagePullSecrets

> **Full question:** Create a docker-registry (image-pull) Secret and reference it via imagePullSecrets; explain when it's required.

```bash
kubectl create secret docker-registry regcred \
  --docker-server=... --docker-username=... --docker-password=...
```
Reference it: `spec.imagePullSecrets: [{ name: regcred }]`.
**Verify:** `kubectl get secret regcred` (type `kubernetes.io/dockerconfigjson`); the pod pulls successfully.
*Why:* **required to pull from a private registry** (or for authenticated Docker Hub pulls, Q2).

<a id="lo4-37"></a>
### Q37 · Secret with several keys — mount only one

> **Full question:** Create a Secret with several keys and selectively mount only one of them into a pod.

```yaml
      volumes:
      - name: sec
        secret:
          secretName: multi
          items:
          - { key: password, path: db-password }   # only this key
```
**Verify:** `kubectl exec <pod> -- ls /etc/sec` shows only `db-password`.
*Why:* `items` selects specific keys and maps them to chosen filenames (least privilege).

<a id="lo4-38"></a>
### Q38 · ClusterIP Service + resolve by DNS from another pod

> **Full question:** Create a ClusterIP Service for a Deployment and resolve it by DNS from another pod (nslookup <svc>.<ns>.svc.cluster.local).

```bash
kubectl expose deployment web --port=80                  # ClusterIP
kubectl run tmp --rm -it --image=busybox -- nslookup web.default.svc.cluster.local
```
**Verify:** nslookup returns the Service's ClusterIP.
*Why:* a ClusterIP Service is reachable cluster-internally by `web.<ns>.svc.cluster.local` (short `web` works within the same namespace).

<a id="lo4-39"></a>
### Q39 · NodePort Service + reach it

> **Full question:** Create a NodePort Service and reach it via minikube service <svc> --url and $(minikube ip):<nodePort>.

```bash
kubectl expose deployment web --type=NodePort --port=80
minikube service web --url
curl $(minikube ip):<nodePort>
```
**Verify:** the URL/curl returns the app's response.
*Why:* NodePort opens a port (30000–32767) on every node; `minikube service --url` prints the reachable URL.

<a id="lo4-40"></a>
### Q40 · LoadBalancer + why EXTERNAL-IP is pending on minikube

> **Full question:** Create a LoadBalancer Service, explain why EXTERNAL-IP is <pending> on minikube, and obtain access with minikube tunnel.

```bash
kubectl expose deployment web --type=LoadBalancer --port=80
kubectl get svc web         # EXTERNAL-IP <pending>
minikube tunnel             # run in a SEPARATE terminal → assigns an IP
```
**Verify:** after `minikube tunnel`, `kubectl get svc web` shows an EXTERNAL-IP you can curl.
*Why:* a LoadBalancer needs a **cloud provider** to allocate the IP; minikube has none, so it stays `<pending>` until `minikube tunnel` simulates one.

<a id="lo4-41"></a>
### Q41 · Headless Service (clusterIP: None) for a StatefulSet

> **Full question:** Create a headless Service (clusterIP: None) for a StatefulSet and show it returns per-pod A records instead of a single VIP.

```yaml
spec: { clusterIP: None, selector: { app: redis }, ports: [{ port: 6379 }] }
```
**Verify:** `kubectl run tmp --rm -it --image=busybox -- nslookup redis.default.svc.cluster.local` returns **one A record per ready pod** (not a single VIP).
*Why:* with no cluster VIP, DNS hands back per-pod addresses → direct addressing (`redis-0.redis...`); this is how StatefulSet clients reach individual members.

<a id="lo4-42"></a>
### Q42 · Service Endpoints / EndpointSlice — how they're populated

> **Full question:** Inspect a Service's Endpoints/EndpointSlice and explain how they're populated from the pod selector.

```bash
kubectl get endpoints web
kubectl get endpointslice -l kubernetes.io/service-name=web
```
**Verify:** the endpoints list the IPs of the Ready pods matching the selector.
*Why:* Kubernetes watches pods matching the Service **selector** and lists the **Ready** ones. **Empty endpoints = selector matches nothing or no pod is Ready** — the #1 cause of "Service returns nothing" (LO5 #25).

<a id="lo4-43"></a>
### Q43 · Cross-namespace access via FQDN (short name fails)

> **Full question:** Demonstrate cross-namespace access using the FQDN, and show the short name fails from another namespace.

```bash
# from a pod in another namespace:
nslookup web.prod.svc.cluster.local      # works
nslookup web                             # fails
```
**Verify:** FQDN resolves; the short name does not.
*Why:* short names resolve only within the **same namespace**; cross-namespace needs the **FQDN** `<svc>.<ns>.svc.cluster.local`.

<a id="lo4-44"></a>
### Q44 · port-forward to a Service vs to a Pod

> **Full question:** Compare kubectl port-forward to a Service versus to a Pod — explain the difference and when each fits.

```bash
kubectl port-forward svc/web 8080:80     # via the Service (picks a backing pod)
kubectl port-forward pod/web-xyz 8080:80 # to one specific pod
```
**Verify:** `curl localhost:8080` while the command runs.
*Why:* Service port-forward tests the service abstraction; pod port-forward targets one exact pod for debugging. Both are local-only tunnels for the command's duration.

<a id="lo4-45"></a>
### Q45 · port vs targetPort vs nodePort

> **Full question:** Explain the difference between a Service's port, targetPort, and nodePort.

- **port** — the port the Service exposes to in-cluster clients.
- **targetPort** — the **container** port traffic is forwarded to.
- **nodePort** — the external port opened on each node (NodePort/LoadBalancer only, 30000–32767).

Traffic path: client → `port` → `targetPort` (pod); externally → `nodePort` → `port` → `targetPort`.

<a id="lo4-46"></a>
### Q46 · Verify ClusterIP connectivity from a throwaway debug pod

> **Full question:** Verify connectivity to a ClusterIP Service from a throwaway debug pod (kubectl run tmp --rm -it --image=busybox -- sh) with wget/nc.

```bash
kubectl run tmp --rm -it --image=busybox -- sh
# inside:
wget -qO- web.default.svc.cluster.local
nc -zv web 80
```
*Why:* `--rm` auto-deletes the pod on exit — the standard way to test in-cluster reachability without touching real workloads.

<a id="lo4-47"></a>
### Q47 · Two Deployments, one Service load-balancing across both

> **Full question:** Create two Deployments and one Service that load-balances across both by sharing a common label; prove requests hit both.

Give both Deployments' pods a **common label** (e.g. `app: shop`) and select on it:
```yaml
selector: { app: shop }
```
**Verify:** `kubectl get endpoints <svc>` lists pods from **both** Deployments; curl repeatedly from a debug pod and observe different pod hostnames.
*Why:* endpoints include every pod matching the selector, so traffic spreads across both.

<a id="lo4-48"></a>
### Q48 · Pod can't reach a Service — systematic diagnosis

> **Full question:** Systematically diagnose why a pod can't reach a Service: DNS → endpoints → selector labels → readiness → port.

```bash
1. DNS:        kubectl run tmp --rm -it --image=busybox -- nslookup <svc>
2. Endpoints:  kubectl get endpoints <svc>          # empty?
3. Selectors:  do Service selector labels == pod labels?
4. Readiness:  are pods Ready? (failing probe → no endpoints)
5. Port:       does targetPort match the container's actual port?
```
*Why:* walk it top-down; the broken link is almost always empty endpoints (selector mismatch) or a failing readiness probe.

<a id="lo4-49"></a>
### Q49 · Expose a service with NodePort and verify

> **Full question:** Expose a service with a NodePort and verify it works.

```bash
kubectl expose deployment web --type=NodePort --port=80
kubectl get svc web -o wide                  # note the nodePort
curl $(minikube ip):<nodePort>               # verify it answers
```

<a id="lo4-50"></a>
### Q50 · HTTP livenessProbe on / port 80 (nginx) + verify

> **Full question:** Add an HTTP livenessProbe hitting / on port 80 to an nginx Deployment and verify via kubectl describe pod.

**Method B — edit:** under the container in `kubectl edit deployment/web` (save & quit, CS-0):
```yaml
        livenessProbe:
          httpGet: { path: /, port: 80 }
          initialDelaySeconds: 5
          periodSeconds: 10
```
**Verify:** `kubectl describe pod <p>` shows the `Liveness` line and any restarts.
*Why:* a failing liveness probe **restarts** the container.

<a id="lo4-51"></a>
### Q51 · tcpSocket readiness probe on redis 6379 + verify

> **Full question:** Add a tcpSocket readiness probe to a redis container on port 6379 and verify it.

```yaml
        readinessProbe:
          tcpSocket: { port: 6379 }
          periodSeconds: 5
```
**Verify:** `kubectl describe pod <p>` shows the `Readiness` line; pod becomes Ready only after the probe passes.
*Why:* a failing **readiness** probe doesn't restart the pod — it removes it from Service endpoints so it stops receiving traffic until it passes.

<a id="lo4-52"></a>
### Q52 · Startup probe for a ~2-minute boot + justify the numbers

> **Full question:** Configure a startup probe with a failureThreshold * periodSeconds budget for a ~2-minute boot and justify the numbers.

```yaml
        startupProbe:
          httpGet: { path: /, port: 80 }
          failureThreshold: 30
          periodSeconds: 5            # 30 × 5 = 150s budget > 120s boot
```
**Verify:** `kubectl describe pod <p>` shows the `Startup` line; the pod isn't killed during the slow boot.
*Why:* budget = `failureThreshold × periodSeconds` must exceed worst-case boot (150s for a ~120s boot, with margin). While the startup probe runs, liveness/readiness are **suspended**.

<a id="lo4-53"></a>
### Q53 · Default behavior when no probes are defined

> **Full question:** Explain the default behavior when no probes are defined at all.

With **no probes**, the container is **Ready as soon as its process starts**, and is only restarted if that process **exits**. Kubernetes can't tell whether the app is actually serving — a hung-but-running process is treated as healthy and keeps receiving traffic. This is why explicit liveness/readiness probes matter.

---

*End of LO4.*

<a id="lo5--solve-problems-with-application-shipping-by-using-containers"></a>
# LO5 — Solve problems with application shipping by using containers

Troubleshooting. Broken **podman commands** (#1–8): run it → see the failure → fix → why.
Broken **Containerfiles** (#10–18): the broken file → mistake → fixed file → how to build &
check. **Kubernetes diagnosis** (#19–43): symptom → diagnose → cause → fix.

<a id="lo5-cheatsheets"></a>
## Cheat-sheets

### CS-5a · The diagnostic trio (use on almost every k8s problem)
```bash
kubectl describe pod <p>                 # Events at the bottom = what went wrong
kubectl logs <p> --previous              # logs of the LAST (crashed) container
kubectl get events --sort-by=.lastTimestamp   # timeline of everything
kubectl get pod <p> -o yaml              # status.containerStatuses → state / lastState / reason
```

### CS-5b · Pod status → likely cause
- **Pending** → can't schedule: insufficient cpu/memory, nodeSelector/taint mismatch, unbound PVC.
- **ImagePullBackOff / ErrImagePull** → bad image name/tag, or private registry without pull secret.
- **CrashLoopBackOff** → starts then exits repeatedly: bad command/config, app error, or no long-running process.
- **OOMKilled** (in lastState) → exceeded its memory limit.
- **Completed** (with restarts) → the command finished; nothing keeps it running.
- **Running but never Ready** → failing readiness probe → no Service endpoints.
- **Terminating (stuck)** → finalizers + grace period.

### CS-5c · podman build & run checks
```bash
podman build -t myimg .                  # build from Containerfile in current dir
podman run -d --name c myimg             # run it
podman logs c                            # why did it exit?
podman ps -a                             # see Exited status / codes
podman inspect c --format '{{.State.Status}}'
```
- `-p HOST:CONTAINER` — order matters; CONTAINER = the port the app actually listens on.
- `--network host` → `-p` is ignored. Bind mount on SELinux → add `:Z`. Name resolution between containers → only on a **user-defined network**.

---

<a id="lo5-index"></a>
## LO5 — Question index

**Broken podman commands:** [1](#lo5-1) [2](#lo5-2) [3](#lo5-3) [4](#lo5-4) [5](#lo5-5) [6](#lo5-6) [7](#lo5-7) [8](#lo5-8)
**Broken Containerfiles:** [10](#lo5-10) [11](#lo5-11) [12](#lo5-12) [13](#lo5-13) [14](#lo5-14) [16](#lo5-16) [17](#lo5-17) [18](#lo5-18)
**Kubernetes diagnosis:** [19](#lo5-19) [20](#lo5-20) [21](#lo5-21) [22](#lo5-22) [23](#lo5-23) [24](#lo5-24) [25](#lo5-25) [26](#lo5-26) [27](#lo5-27) [28](#lo5-28) [29](#lo5-29) [30](#lo5-30) [31](#lo5-31) [32](#lo5-32) [33](#lo5-33) [34](#lo5-34) [35](#lo5-35) [36](#lo5-36) [37](#lo5-37) [38](#lo5-38) [39](#lo5-39) [40](#lo5-40) [41](#lo5-41) [42](#lo5-42) [43](#lo5-43)

---

<a id="lo5-1"></a>
### Q1 · Reversed port mapping

> **Full question:** `podman run -d --name web -p 80:8080 docker.io/library/nginx` — nginx listens on 80; the site isn't reachable on the host port you expected. What's reversed?

**See the problem:** `curl localhost:80` → nothing. `podman port web` shows 80→8080.
**Fix:** `podman run -d --name web -p 8080:80 docker.io/library/nginx`, then `curl localhost:8080`.
**Why:** `-p` is `HOST:CONTAINER`. The original sent host 80 → container **8080**, but nginx listens on **80**, so nothing answered. The container-side port must match the app's listening port.

<a id="lo5-2"></a>
### Q2 · MySQL root password with no value

> **Full question:** `podman run -d --name db -e MYSQL_ROOT_PASSWORD docker.io/library/mysql` — the container exits during init. Inspect podman logs.

**See the problem:** `podman logs db` → complains the root password is unset; `podman ps -a` shows db Exited.
**Fix:** `podman run -d --name db -e MYSQL_ROOT_PASSWORD=secret docker.io/library/mysql`
**Why:** `-e VAR` with no `=value` passes the host's variable (empty/unset). MySQL refuses to init without `MYSQL_ROOT_PASSWORD` (or `MYSQL_ALLOW_EMPTY_PASSWORD=1` / `MYSQL_RANDOM_ROOT_PASSWORD=1`).

<a id="lo5-3"></a>
### Q3 · -p ignored with --network host

> **Full question:** `podman run -d --name app --network host -p 8080:80 docker.io/library/nginx` — why is the -p flag effectively ignored here?

**See the problem:** nginx is reachable on host **80**, not on 8080; podman may even warn that `-p` is ignored with host networking.
**Fix:** drop one — `--network host` (reach nginx on host 80) **or** default network with `-p 8080:80`.
**Why:** `--network host` shares the host's network namespace, so there's nothing to publish into; port publishing only applies to isolated container networks.

<a id="lo5-4"></a>
### Q4 · busybox exits immediately

> **Full question:** `podman run -d --name c1 docker.io/library/busybox` — the container goes straight to Exited (0). Why, and how do you keep it running?

**See the problem:** `podman ps -a` → c1 Exited (0).
**Fix:** `podman run -d --name c1 docker.io/library/busybox sleep infinity`, then `podman ps` shows it Up.
**Why:** busybox's default command exits at once with no input; a container lives only as long as its main process — give it a long-running foreground process.

<a id="lo5-5"></a>
### Q5 · --rm + detached loses the logs

> **Full question:** `podman run --rm -d --name job docker.io/library/alpine echo hello` — then podman logs job fails. Explain the --rm + detached gotcha.

**See the problem:** `podman logs job` → "no such container" (it's already gone).
**Fix:** drop `--rm` if you need output: `podman run -d --name job alpine echo hello; podman logs job`. Or run attached (no `-d`) to see it live.
**Why:** `echo hello` finishes instantly and `--rm` auto-deletes the container (and its logs) the moment it stops.

<a id="lo5-6"></a>
### Q6 · Memory limit too low for MySQL

> **Full question:** `podman run -d -p 8080:80 --memory 8m docker.io/library/mysql` — the database never becomes healthy. What limit is the problem?

**See the problem:** `podman logs <c>` shows it dying during init / OOM; it never reports healthy.
**Fix:** raise it, e.g. `--memory 512m`. (Also note `-p 8080:80` targets the wrong port — MySQL listens on 3306.)
**Why:** 8m is far below MySQL's startup needs; it's killed before it can initialize.

<a id="lo5-7"></a>
### Q7 · No name resolution on the default network

> **Full question:** Two alpine containers `a` and `b` with `sleep 1d`, then `podman exec a ping b` — name resolution fails. Why, on the default network, and how do you fix it?

**See the problem:** `podman exec a ping b` → "bad address 'b'".
**Fix:**
```bash
podman network create mynet
podman run -d --name a --network mynet alpine sleep 1d
podman run -d --name b --network mynet alpine sleep 1d
podman exec a ping b        # resolves now
```
**Why:** the default network has no name DNS; only user-defined networks run the resolver (aardvark-dns). Put both on the same user-defined network.

<a id="lo5-8"></a>
### Q8 · Missing SELinux relabel flag on bind mount

> **Full question:** `podman run -d --name web -v ./html:/usr/share/nginx/html docker.io/library/nginx` — the files aren't visible / SELinux denies access. What mount flag is missing?

**See the problem:** the page is empty / 403; `podman logs web` or `ausearch` shows SELinux AVC denials.
**Fix:** `podman run -d --name web -v ./html:/usr/share/nginx/html:Z docker.io/library/nginx`
**Why:** on SELinux-enforcing hosts (RHEL/Fedora) a bind mount needs a relabel flag. `:Z` = private label for this container (`:z` = shared between containers). Without it the container can't read the host dir.

<a id="lo5-9"></a>
### Q9 · *(section divider — not a standalone question)*
The list switches to **Containerfile/Dockerfile** problems (#10–18): for each, identify the mistake, fix it, explain. Build & check pattern: `podman build -t test .` then `podman run --rm test`.

<a id="lo5-10"></a>
### Q10 · apt-get install without update

> **Full question:** `FROM debian:12` / `RUN apt-get install -y nginx` — the build fails to find the package. What's missing before install?

**Broken:** no `apt-get update`, so package lists are empty and the package isn't found.
**Fixed:**
```dockerfile
FROM debian:12
RUN apt-get update && apt-get install -y nginx
```
**Build & check:** `podman build -t t .` succeeds; `apt-get update` must share the same RUN as install so caching can't leave stale lists.

<a id="lo5-11"></a>
### Q11 · Shell-form CMD breaks signal handling

> **Full question:** `FROM python:3.11` ... `CMD python app.py` — the app starts but can't be stopped cleanly / doesn't handle signals. Shell vs exec form?

**Broken:** `CMD python app.py` is **shell form** → runs as `/bin/sh -c "python app.py"`, so `sh` is PID 1 and gets SIGTERM; python never does.
**Fixed:**
```dockerfile
CMD ["python", "app.py"]
```
**Check:** `podman stop <c>` now stops promptly. **Exec form** makes python PID 1, so it receives signals and shuts down cleanly.

<a id="lo5-12"></a>
### Q12 · Layer caching: copy deps before source

> **Full question:** `FROM node:20` / `COPY . /app` / `WORKDIR /app` / `RUN npm install` — every source change triggers a full npm install. Reorder for layer caching.

**Broken:** `COPY . /app` before `npm install` busts the cache on any source change.
**Fixed:**
```dockerfile
FROM node:20
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
```
**Check:** rebuild after a source-only edit → npm install layer is reused (CACHED). Deps re-install only when `package*.json` changes.

<a id="lo5-13"></a>
### Q13 · Overwriting PATH

> **Full question:** `FROM alpine:3.20` / `ENV PATH=/app/bin` / `RUN apk add --no-cache curl` — afterwards curl/sh aren't found. What did setting PATH break?

**Broken:** `ENV PATH=/app/bin` **replaces the whole PATH**, dropping `/usr/bin`, `/bin`, so `sh`, `curl`, `apk` aren't found.
**Fixed:**
```dockerfile
ENV PATH="/app/bin:$PATH"
```
**Why:** always **append** to PATH, never overwrite it.

<a id="lo5-14"></a>
### Q14 · EXPOSE ≠ what the app listens on (combined with Q15)

> **Full question (14+15):** `FROM ubuntu:24.04` / `EXPOSE 8080` / `CMD ["python3","-m","http.server","3000"]` — you map `-p 8080:8080` but nothing answers. What mismatch is there, and what does EXPOSE actually do?

**Broken:** the server listens on **3000**, but you `EXPOSE 8080` and map `8080:8080` → traffic hits a port nothing listens on.
**Fixed:** make ports consistent — either run on 8080 (`http.server 8080` + `-p 8080:8080`) or map `-p 8080:3000`.
**Why:** **EXPOSE is documentation only** — it doesn't publish a port or change what the app binds to. Only `-p` publishes, and the container port must match the listening port.

<a id="lo5-16"></a>
### Q16 · Cut image size in the same layer

> **Full question:** `FROM debian:12` / `RUN apt-get update && apt-get install -y build-essential` — the image is huge. How do you cut the size in the same layer?

**Broken:** the apt cache (and recommended packages) stay in the layer, bloating the image.
**Fixed:**
```dockerfile
RUN apt-get update \
 && apt-get install -y --no-install-recommends build-essential \
 && rm -rf /var/lib/apt/lists/*
```
**Check:** `podman images` shows a smaller size. Cleanup must be in the **same RUN/layer** — deleting cache in a later layer doesn't shrink the earlier one (layers are additive).

<a id="lo5-17"></a>
### Q17 · USER set too early → permission denied

> **Full question:** `FROM python:3.11` / `USER appuser` / `COPY requirements.txt /app/...` / `RUN pip install -r ...` — permission denied. What's wrong with the USER/COPY ordering and ownership?

**Broken:** switching to `appuser` before COPY/install means the non-root user can't write to `/app` or install packages, and COPY'd files are root-owned.
**Fixed:**
```dockerfile
FROM python:3.11
WORKDIR /app
COPY --chown=appuser:appuser requirements.txt .
RUN pip install -r requirements.txt
USER appuser            # switch AFTER privileged steps
```
**Why:** do root-needing work first, set ownership with `COPY --chown`, then drop to the non-root user.

<a id="lo5-18"></a>
### Q18 · Multi-stage build to shrink a Go image

> **Full question:** `FROM golang:1.22` / `COPY . /src` / `RUN go build -o app .` / `CMD ["/src/app"]` — the final image is ~1 GB. How would a multi-stage build fix it?

**Broken:** the final image carries the whole Go toolchain.
**Fixed:**
```dockerfile
FROM golang:1.22 AS build
WORKDIR /src
COPY . .
RUN go build -o /app .

FROM alpine:3.20          # or scratch
COPY --from=build /app /app
CMD ["/app"]
```
**Check:** `podman images` → from ~1 GB to tens of MB. Build in one stage, copy **only the binary** into a minimal final image.

<a id="lo5-19"></a>
### Q19 · Pod stuck Pending

> **Full question:** A pod is stuck Pending. Walk through the diagnosis with kubectl describe pod and identify the reason from the events (scheduling / resources / PVC / nodeSelector).

**Diagnose:** `kubectl describe pod <p>` → read **Events**.
**Causes & fixes:** Insufficient cpu/memory → lower requests or add a node; nodeSelector/taint with no match → label the node or remove the selector; unbound PVC → create the PVC/StorageClass.
**Why:** Pending = the scheduler couldn't place the pod; the Events line names the failed constraint.

<a id="lo5-20"></a>
### Q20 · Broken YAML indentation

> **Full question:** Find a deployment from older tasks, remove a couple of spaces and try to deploy it. Identify the cause in the error messages and fix it.

**Diagnose:** `kubectl apply -f deploy.yaml` → "error converting YAML to JSON" or "field X: invalid type".
**Fix:** restore correct indentation; pre-check with `kubectl apply -f deploy.yaml --dry-run=client` (Q40).
**Why:** YAML is whitespace-sensitive — a missing/extra space nests a field under the wrong parent or parses it as the wrong type.

<a id="lo5-21"></a>
### Q21 · ImagePullBackOff

> **Full question:** A pod is in ImagePullBackOff. Diagnose the cause (image typo, missing tag, private registry without pull secret) and fix it.

**Diagnose:** `kubectl describe pod <p>` → Events show the pull error.
**Cause & fix:** typo → correct the image/tag; non-existent tag → use a real one; private registry → create a docker-registry Secret + `imagePullSecrets` (LO4 #36).
**Why:** the kubelet can't pull; the event text says which of the three it is.

<a id="lo5-22"></a>
### Q22 · CrashLoopBackOff

> **Full question:** A pod is in CrashLoopBackOff. Use kubectl logs --previous to read the last crash and fix the root cause.

**Diagnose:** `kubectl logs <p> --previous` (last crashed container) + `kubectl describe pod <p>` (restart count, exit code).
**Cause & fix:** bad command/args, missing env/config, startup error, or no long-running process (Q29). Fix the underlying error.
**Why:** the container starts, exits, and is restarted with growing back-off — an app/config failure, not a missing image.

<a id="lo5-23"></a>
### Q23 · OOMKilled

> **Full question:** A pod's last state shows OOMKilled. Confirm it from kubectl get pod -o yaml / describe, then fix it (raise the memory limit or fix the app).

**Diagnose:** `kubectl get pod <p> -o yaml` → `status.containerStatuses[].lastState.terminated.reason: OOMKilled` (also in `describe`).
**Fix:** raise the container's memory **limit**, or fix the leak/over-allocation.
**Why:** the container exceeded its memory limit and the kernel killed it.

<a id="lo5-24"></a>
### Q24 · Pods never become Ready (readiness probe)

> **Full question:** A Deployment's pods never become Ready. Trace it to a failing readiness probe and correct the probe or the app.

**Diagnose:** `kubectl describe pod <p>` → "Readiness probe failed".
**Fix:** correct the probe (right path/port, longer `initialDelaySeconds`) or fix the app so it serves on that path/port.
**Why:** a failing readiness probe keeps the pod out of Service endpoints → Running but never Ready.

<a id="lo5-25"></a>
### Q25 · Service returns nothing (label mismatch)

> **Full question:** A Service returns nothing. Diagnose a Service/pod label mismatch using kubectl get endpoints.

**Diagnose:** `kubectl get endpoints <svc>` → **empty** = no pods match.
**Fix:** make the Service `selector` equal the pods' labels.
**Why:** endpoints come only from selector-matching, Ready pods; no match → no backends → no response.

<a id="lo5-26"></a>
### Q26 · selector.matchLabels ≠ template labels

> **Full question:** Given a manifest where spec.selector.matchLabels doesn't match spec.template.metadata.labels, explain the error kubectl apply returns and fix it.

**Diagnose:** `kubectl apply` → "`selector` does not match template `labels`".
**Fix:** make `spec.selector.matchLabels` and `spec.template.metadata.labels` identical.
**Why:** a Deployment must select the pods it creates; mismatched labels would orphan them, so the API rejects it.

<a id="lo5-27"></a>
### Q27 · Requests exceed node capacity

> **Full question:** Given a manifest whose resources.requests exceed any node's capacity, explain why the pod is Pending (Insufficient cpu/memory) and fix it.

**Diagnose:** `kubectl describe pod <p>` → "0/N nodes available: Insufficient cpu/memory".
**Fix:** lower `resources.requests` to fit a node, or add a bigger node.
**Why:** the scheduler needs a node with the requested resources free; none has them → Pending.

<a id="lo5-28"></a>
### Q28 · Pod mounts a non-existent PVC

> **Full question:** Given a pod mounting a PVC that doesn't exist, diagnose the FailedMount/Pending and create the PVC.

**Diagnose:** `kubectl describe pod <p>` → "persistentvolumeclaim ... not found" / FailedMount; pod Pending.
**Fix:** create the PVC (and matching StorageClass/PV) first.
**Why:** the pod can't start until its volume claim binds to a volume.

<a id="lo5-29"></a>
### Q29 · ubuntu with no long-running command

> **Full question:** A container based on ubuntu with no long-running command shows Completed/CrashLoopBackOff. Explain why and add a proper command.

**Diagnose:** the default shell exits immediately → **Completed**; with `restartPolicy: Always` it loops → **CrashLoopBackOff**.
**Fix:** give it a foreground process, e.g. `command: ["sleep","infinity"]` or the real app.
**Why:** a container stays up only while its main process runs; a base OS image has no long-running entrypoint.

<a id="lo5-30"></a>
### Q30 · Triage a pod's history over time

> **Full question:** Use kubectl get events --sort-by=.lastTimestamp (or kubectl events) to triage everything that happened to a pod over time.

```bash
kubectl get events --sort-by=.lastTimestamp
kubectl events --for pod/<p>            # newer syntax, scoped to one object
```
**Why:** events are time-ordered breadcrumbs (scheduling, pulls, probe failures, OOM) — the fastest overview of what happened and when.

<a id="lo5-31"></a>
### Q31 · Debug DNS from inside a pod

> **Full question:** Use kubectl exec -it to get a shell in a pod and debug DNS with nslookup and cat /etc/resolv.conf.

```bash
kubectl exec -it <p> -- sh
nslookup <service>          # does it resolve?
cat /etc/resolv.conf        # nameserver = cluster DNS (CoreDNS) IP
```
**Why:** confirms DNS works and the pod points at cluster DNS; a wrong `search`/`nameserver` explains name failures.

<a id="lo5-32"></a>
### Q32 · Ephemeral debug container (distroless/no-shell)

> **Full question:** Use an ephemeral debug container (kubectl debug -it <pod> --image=busybox --target=<container>) to troubleshoot a no-shell/distroless container.

```bash
kubectl debug -it <pod> --image=busybox --target=<container>
```
**Why:** distroless/no-shell containers have no `sh` to exec into; `kubectl debug` attaches a temporary busybox sharing the target's namespaces so you can inspect it.

<a id="lo5-33"></a>
### Q33 · Throwaway pod to test Service connectivity

> **Full question:** Spin up a throwaway kubectl run tmp --rm -it --image=busybox pod to test connectivity to a Service from inside the cluster.

```bash
kubectl run tmp --rm -it --image=busybox -- sh
# inside: wget -qO- <svc>.<ns>.svc.cluster.local   /   nc -zv <svc> <port>
```
**Why:** `--rm` cleans up on exit; tests reachability from inside the cluster without touching real workloads.

<a id="lo5-34"></a>
### Q34 · Node NotReady

> **Full question:** A node shows NotReady. List what you'd check (kubelet, disk pressure, CNI) and inspect it on minikube with kubectl describe node and sudo minikube logs.

**Check:** kubelet running/healthy; node conditions (DiskPressure, MemoryPressure); the CNI/network plugin.
```bash
kubectl describe node <node>      # Conditions + Events
sudo minikube logs
```
**Why:** NotReady = the kubelet stopped reporting healthy — usually a crashed kubelet, resource pressure, or broken networking.

<a id="lo5-35"></a>
### Q35 · Pod stuck Terminating

> **Full question:** A pod is stuck Terminating. Explain finalizers and the grace period, then force-delete it (--grace-period=0 --force) and state the risks.

```bash
kubectl delete pod <p> --grace-period=0 --force
```
**Why:** a pod won't disappear while **finalizers** are pending or during the **grace period**. Force-delete removes it from the API immediately — risk: the underlying process/volume may still be active, so you can get data corruption or a "ghost" workload, especially with StatefulSets.

<a id="lo5-36"></a>
### Q36 · Wrong apiVersion/kind pairing

> **Full question:** Given a manifest with the wrong apiVersion/kind pairing (e.g. apps/v1 + Pod), explain the validation error and correct the group/version.

**Diagnose:** `kubectl apply` → "no matches for kind Pod in version apps/v1".
**Fix:** Pod is in core `v1` → `apiVersion: v1`; use `apps/v1` for Deployment/ReplicaSet/StatefulSet/DaemonSet.
**Why:** each kind lives in a specific API group/version; mismatched pairs don't exist.

<a id="lo5-37"></a>
### Q37 · CreateContainerConfigError (missing ConfigMap key)

> **Full question:** A pod fails with CreateContainerConfigError because a configMapKeyRef key is missing. Diagnose and fix.

**Diagnose:** `kubectl describe pod <p>` → "couldn't find key <k> in ConfigMap".
**Fix:** add the key to the ConfigMap, or correct the `configMapKeyRef.key`.
**Why:** the kubelet can't build the container's env because the referenced key doesn't exist, so it never starts.

<a id="lo5-38"></a>
### Q38 · Secret in a different namespace

> **Full question:** A pod can't mount a Secret that lives in a different namespace. Explain namespace scoping and fix it.

**Diagnose:** mount fails / "secret not found"; the Secret exists, but in another namespace.
**Fix:** create the Secret in the **pod's own namespace**.
**Why:** Secrets and ConfigMaps are **namespace-scoped** — a pod can only reference ones in its own namespace; there's no cross-namespace mount.

<a id="lo5-39"></a>
### Q39 · ProgressDeadlineExceeded

> **Full question:** A rollout is stuck with ProgressDeadlineExceeded. Use kubectl describe deployment / rollout status to find the cause and remediate.

**Diagnose:** `kubectl rollout status deployment/<d>` + `kubectl describe deployment <d>` → new ReplicaSet not progressing.
**Cause & fix:** new pods can't become available — bad image (Q21), crash (Q22), or failing probe (Q24). Fix root cause or `kubectl rollout undo`.
**Why:** the Deployment didn't reach desired state within `progressDeadlineSeconds` (default 600s).

<a id="lo5-40"></a>
### Q40 · Catch field typos before applying

> **Full question:** Catch indentation/field typos (e.g. imagePullpolicy, misnested ports) before applying with kubectl apply --dry-run=server / --validate=true, then fix them.

```bash
kubectl apply -f m.yaml --dry-run=server     # server-side schema validation
```
**Fix:** correct field names (`imagePullPolicy`, not `imagePullpolicy`) and nesting.
**Why:** `--dry-run=server` validates against the real API schema without creating anything; client-side often silently drops unknown fields, so server-side is the reliable check.

<a id="lo5-41"></a>
### Q41 · App can't reach its database Service

> **Full question:** An app's logs say it can't reach its database Service. Verify in order: pod running → Service exists → endpoints populated → DNS resolves → correct port — and identify the broken link.

**Diagnose (in order):**
```bash
kubectl get pod <db>                     # 1. DB pod Running/Ready?
kubectl get svc <db-svc>                 # 2. Service exists?
kubectl get endpoints <db-svc>           # 3. endpoints populated? (empty = selector/readiness)
kubectl run tmp --rm -it --image=busybox -- nslookup <db-svc>   # 4. DNS resolves?
# 5. is the app using the Service's correct port?
```
**Why:** walking the chain pinpoints the broken link — usually empty endpoints (selector mismatch / not Ready) or a wrong port.

<a id="lo5-42"></a>
### Q42 · Read state/lastState/restartCount

> **Full question:** Read a container's state/lastState and restartCount with kubectl get pod -o jsonpath to determine the root cause of repeated restarts.

```bash
kubectl get pod <p> -o jsonpath='{.status.containerStatuses[0].lastState.terminated.reason}{"\n"}'
kubectl get pod <p> -o jsonpath='{.status.containerStatuses[0].restartCount}{"\n"}'
```
**Why:** `lastState.terminated.reason` (e.g. OOMKilled, Error) + restart count tells you why a container keeps dying without scrolling describe output.

<a id="lo5-43"></a>
### Q43 · OpenShift Route vs Ingress

> **Full question:** Compare OpenShift Route to Ingress.

**Ingress** is the Kubernetes-native object that routes external HTTP(S) to Services by host/path, but it needs a **separately installed Ingress controller** (nginx, Traefik…). **Route** is OpenShift's built-in equivalent: the integrated HAProxy router is already present, so there's no controller to install, and TLS modes (edge / passthrough / re-encrypt) and host setup are simpler. In short, a Route is OpenShift's pre-wired, easier version of Ingress-plus-controller. *(See LO6 CS-C.)*

---

*End of LO5.*

<a id="lo6--evaluate-the-use-of-selected-container-orchestration-systems-theory"></a>
# LO6 — Evaluate the use of selected container orchestration systems (theory)

This LO is essay-style: each question wants a clear **standing** (your position) plus
**arguments**. The same comparisons keep coming back, so the recurring ones are summarized
once in the cheat-sheets below (kept as bullets for quick scanning); each question then gives
a self-contained prose answer you can write down directly.

<a id="lo6-cheatsheets"></a>
## Cheat-sheets (shared comparisons)

### CS-A · Docker vs Podman
- **Architecture:** Docker has a central, long-running **root daemon** (`dockerd`); every CLI command talks to it. Podman is **daemonless** — each command is its own short-lived process (fork/exec model), no background service to keep alive.
- **Rootless:** Podman runs **rootless by default**: containers map to an unprivileged user via Linux **user namespaces**, so a container "root" is a normal host user. Docker can do rootless too but it's the daemon's traditional weak point — historically the daemon ran as root.
- **Security takeaway:** no privileged daemon = smaller attack surface and no single root-owned process that a container escape lands on. Integrates cleanly with **systemd** (a container = a service).
- **Compatibility:** Podman CLI is Docker-compatible (`alias docker=podman` mostly works); both build OCI images.

### CS-B · Kubernetes vs OpenShift
- **What OpenShift is:** Red Hat's enterprise Kubernetes **distribution** — k8s plus integrated tooling, opinionated defaults, and commercial support.
- **Security/hardening:** OpenShift is **secure-by-default** — Security Context Constraints (SCC) block running as root by default, stricter RBAC, an integrated image registry, built-in OAuth. Vanilla k8s is permissive out of the box and you harden it yourself.
- **Developer experience:** OpenShift adds **Source-to-Image (S2I)** (push source → it builds & deploys an image for you), **Routes** (simpler than raw Ingress), a web developer **console**, and integrated CI/CD (Pipelines/Tekton, GitOps/ArgoCD). Plain k8s = `kubectl` + YAML, more manual.
- **Trade-off:** OpenShift = batteries included, vendor support, less to assemble, but **vendor cost + some Red Hat lock-in + more abstraction to learn**. Vanilla k8s = maximum flexibility, no licence cost, but you integrate and operate everything yourself.

### CS-C · Routes vs Ingress *(also LO5 #43)*
- **Ingress** (k8s native): an API object + a separately-installed **Ingress controller** (nginx, Traefik…) that routes external HTTP(S) to Services by host/path.
- **Route** (OpenShift): built-in, no separate controller to install; the integrated **HAProxy router** is already there. Simpler TLS (edge/passthrough/re-encrypt) and host setup.
- **Summary:** Route = OpenShift's pre-wired, easier equivalent of Ingress + controller.

### CS-D · When is full Kubernetes overkill?
Reach for something lighter when scale and team are small:
- **Docker Compose (single host):** one machine, a few services, no HA needed.
- **Docker Swarm:** simple multi-node clustering, far lower learning curve, but a shrinking ecosystem.
- **k3s:** a certified, lightweight k8s (single binary, low memory) for edge / small on-prem — same API, tiny footprint.
- **Full k8s:** justified when you need real HA, autoscaling, large teams, rich ecosystem (operators, service mesh), and multi-node resilience. Overkill for a 2-person MVP on one box.

---

<a id="lo6-index"></a>
## LO6 — Question index

| Q | Topic | Q | Topic |
|---|-------|---|-------|
| [1](#lo6-1) | k8s vs Docker networking | [13](#lo6-13) | Defend OpenShift (integrated platform) |
| [2](#lo6-2) | k8s vs Podman storage | [14](#lo6-14) | k8s vs VM storage provisioning |
| [3](#lo6-3) | Operator pattern vs manifests | [15](#lo6-15) | 2-engineer startup: what to use |
| [4](#lo6-4) | kubectl vs OpenShift S2I (DX) | [16](#lo6-16) | Docker vs Podman (daemonless/rootless) |
| [5](#lo6-5) | Migrate OpenShift → k8s | [17](#lo6-17) | Self-managed vs managed k8s (day-2) |
| [6](#lo6-6) | CI/CD agents: in-cluster vs VMs | [18](#lo6-18) | Spiky global traffic: recommendation |
| [7](#lo6-7) | k8s + cloud / dynamic capacity | [19](#lo6-19) | Outgrown Swarm/Compose → k8s? |
| [8](#lo6-8) | OpenShift vs k8s security | [20](#lo6-20) | k8s for microservices? |
| [9](#lo6-9) | OpenShift vs k8s learning curve | [21](#lo6-21) | k8s for enterprise apps? |
| [10](#lo6-10) | Full k8s vs k3s footprint | [22](#lo6-22) | OpenShift for microservices? |
| [11](#lo6-11) | Compose single-host vs k8s | [23](#lo6-23) | OpenShift for enterprise apps? |
| [12](#lo6-12) | Vendor lock-in: k8s vs OpenShift | | |

---

<a id="lo6-1"></a>
### Q1 · Networking model: Kubernetes vs Docker

> **Full question:** Analyze the networking model of Kubernetes versus Docker's network.
Kubernetes uses a flat, cluster-wide network in which **every pod gets its own routable IP and all pods can reach each other without NAT**, whereas Docker's default model is host-local and bridge/NAT-based. In Docker, containers sit on a per-host `bridge` network with private IPs behind NAT, so to expose them you publish ports with `-p`, and cross-host networking requires an overlay or Swarm; DNS-by-name works only on user-defined networks. In Kubernetes the **CNI** plugin (Calico, Flannel, Cilium…) gives each pod a unique IP in one flat address space, so pods on different nodes talk directly without port publishing. Because pod IPs are ephemeral, stable access to a changing set of pods goes through a **Service** (a virtual IP plus DNS name), and external traffic enters through NodePort, LoadBalancer, or Ingress. The key contrast is that Docker is per-host, NAT-based, and needs manual port publishing, while Kubernetes provides a cluster-wide flat IP space where the Service abstraction decouples clients from pods that come and go.

<a id="lo6-2"></a>
### Q2 · Storage abstraction: Kubernetes vs Podman — which is more flexible for stateful workloads?

> **Full question:** Evaluate Kubernetes storage abstraction (CSI, StorageClasses, dynamic provisioning) against Podman's storage plugins. Which is more flexible for stateful workloads, and why?
Kubernetes is clearly more flexible for stateful workloads, because its storage model is built for dynamic, cluster-wide provisioning while Podman's stops at a single host. In Kubernetes an application requests storage with a **PVC** without knowing the backend, and a **StorageClass** plus a **CSI** driver provisions a real volume on demand (cloud disk, Ceph, NFS, and so on); StatefulSets extend this by giving each replica its own PVC, and you control access modes and reclaim behaviour declaratively. Podman supports named volumes and volume plugins, but it is fundamentally single-host: there is no cluster-wide dynamic provisioning, no automatic per-replica volumes, and no scheduling-aware storage. So while Podman volumes are perfectly good for local or single-node use, the CSI ecosystem combined with dynamic provisioning and StatefulSet-per-replica volumes makes Kubernetes the far stronger choice for distributed stateful services.

<a id="lo6-3"></a>
### Q3 · Operator pattern (CRDs + controllers) vs plain manifests

> **Full question:** Evaluate the operator pattern (CRDs + controllers) for managing stateful services versus using only k8s manifests, in terms of control and effort.
The operator pattern gives you more control and less day-2 effort for complex stateful services, at the price of more upfront work to build or adopt one. With plain manifests you declare desired state once and Kubernetes keeps the pods running, but application-level operations such as backups, failover, version upgrades, or re-sharding a database remain manual scripts and runbooks. An operator combines a **CRD**, which extends the Kubernetes API with a domain object like a `PostgresCluster`, with a **custom controller** that encodes operational knowledge and continuously reconciles that object, automating provisioning, scaling, backups, failover, and upgrades. The trade-off is straightforward: for a stateful service you run at scale an operator pays off enormously by removing operational toil, whereas for a simple stateless app it is unnecessary complexity and plain manifests are enough.

<a id="lo6-4"></a>
### Q4 · Developer experience: plain k8s (kubectl + manifests) vs OpenShift S2I + console

> **Full question:** Assess the developer experience of plain Kubernetes (kubectl + manifests) versus OpenShift's Source-to-Image (S2I) and developer console. What does each optimize for?
Plain Kubernetes optimizes for control and portability, while OpenShift's Source-to-Image optimizes for developer speed and easy onboarding. With plain k8s you work through `kubectl` and YAML, which gives maximum transparency, flexibility, and portability, but the developer has to understand and write Deployments, Services, Ingress, and the rest, carrying a higher cognitive load. OpenShift's **S2I** lets a developer push source code and have the platform build the image and deploy it automatically, with no Dockerfile needed for supported runtimes, backed by a web console, Routes, and an integrated registry and CI. In short, plain k8s suits engineers who want full control and no vendor coupling, while S2I suits getting application developers shipping quickly without deep container or Kubernetes expertise.

<a id="lo6-5"></a>
### Q5 · Critically evaluate migrating a workload OpenShift → Kubernetes

> **Full question:** Critically evaluate migrating a workload from OpenShift to Kubernetes.
Migrating from OpenShift to vanilla Kubernetes is entirely doable, but you should expect to re-implement the conveniences OpenShift gave you for free. The core objects — Deployments, Services, ConfigMaps, Secrets, PVCs — are standard Kubernetes and move cleanly, so the applications themselves are portable. What breaks is the platform layer around them: **Routes** must become **Ingress plus an installed controller**, **S2I builds** must become your own CI pipelines and Dockerfiles, **SCCs** must be replaced by Pod Security and RBAC you configure yourself, and the integrated registry, OAuth, monitoring, and logging stacks all have to be re-provisioned. The reason to migrate is to drop the licence cost and Red Hat coupling and gain portability across any conformant cluster; the cost is that you take on all the operational and security plumbing OpenShift previously handled. The migration effort is therefore real, and it is mostly about rebuilding the platform layer rather than the apps.

<a id="lo6-6"></a>
### Q6 · CI/CD build agents inside Kubernetes vs on dedicated VMs

> **Full question:** Evaluate running CI/CD build agents (Jenkins agents, GitLab runners, Tekton) inside Kubernetes versus on dedicated VMs, considering isolation, autoscaling, and noisy-neighbor effects.
Running build agents inside Kubernetes wins on elasticity and cost, while dedicated VMs win on isolation and predictability. In-cluster agents such as Jenkins agents, GitLab runners, or Tekton tasks run as ephemeral pods spun up per build, which means they autoscale, you pay only for what you use, you avoid idle VMs, and you get consistent containerized build environments. The risks in-cluster are noisy-neighbour effects, where a heavy build starves other pods, and weaker isolation, since builds execute untrusted code next to real workloads; both are mitigated with resource requests and limits, quotas, separate namespaces or nodes, and rootless or sandboxed builders like Kaniko or Buildah. Dedicated VMs give strong isolation and predictable performance but bring idle cost and manual scaling. On balance, in-cluster agents — especially Kubernetes-native Tekton — are the better default for elastic, cloud-native pipelines when paired with resource limits and isolation controls, and you reserve VMs for builds that need strong isolation or special hardware.

<a id="lo6-7"></a>
### Q7 · How does Kubernetes integrate with cloud + dynamic capacity provisioning?

> **Full question:** How does Kubernetes integrate with cloud environments and dynamic capacity provisioning?
Kubernetes integrates with cloud platforms through a cloud-controller-manager and a set of autoscalers that together deliver compute, storage, and load balancing on demand. A **LoadBalancer** Service provisions a real cloud load balancer automatically, and CSI drivers with StorageClasses dynamically provision cloud disks such as EBS or Persistent Disk. For elasticity, the **Cluster Autoscaler** adds or removes whole nodes when pods cannot be scheduled or nodes sit idle, while the **Horizontal** and **Vertical Pod Autoscalers** scale the number of pod replicas or their resource requests based on CPU, memory, or custom metrics. Managed offerings like EKS, GKE, and AKS wire all of this directly to the cloud provider, so capacity scales elastically with demand without manual intervention.

<a id="lo6-8"></a>
### Q8 · OpenShift vs vanilla k8s security/hardening — why regulated industries pick OpenShift

> **Full question:** Critically evaluate the default security and hardening of OpenShift versus vanilla Kubernetes. Why do enterprises in regulated industries often choose OpenShift?
OpenShift is secure-by-default whereas vanilla Kubernetes is permissive and must be hardened by you, and that difference is exactly why regulated enterprises favour OpenShift. Out of the box OpenShift uses **Security Context Constraints** to block containers from running as root, ships with integrated OAuth and fine-grained RBAC, provides a trusted internal image registry, offers FIPS and compliance options, and comes with vendor security support and CVE-patching SLAs. Vanilla Kubernetes ships open: root containers and broad permissions are allowed, and Pod Security has to be configured manually. For industries such as finance, healthcare, and government, that out-of-the-box hardening together with auditability and commercial support satisfies compliance requirements without the organization having to assemble and certify the entire security stack itself.

<a id="lo6-9"></a>
### Q9 · OpenShift vs plain k8s — learning curve / skill set. Does the abstraction help or hinder beginners?

> **Full question:** Critically evaluate the learning curve and required skill set of OpenShift versus plain Kubernetes. Does OpenShift's added abstraction help or hinder a team new to containers?
OpenShift's added abstraction helps application developers but adds a second layer for operators to learn, so the net effect on a team new to containers is positive provided they actually use the guardrails. On the helpful side, the console, S2I, Routes, and secure defaults let newcomers ship without first mastering raw YAML and manual hardening. On the hindering side, OpenShift-specific concepts such as SCCs, Routes, BuildConfigs, and the `oc` CLI sit on top of Kubernetes, so the team still eventually needs the underlying k8s fundamentals and the total surface area to learn is larger. The verdict is that for a team new to containers who wants to become productive quickly and safely, OpenShift's abstractions help, whereas for engineers whose goal is to deeply learn portable Kubernetes, that extra layer can obscure the fundamentals.

<a id="lo6-10"></a>
### Q10 · Full Kubernetes vs k3s footprint for small on-prem — when is full k8s overkill?

> **Full question:** Compare the resource footprint of full Kubernetes versus k3s for a small on-premise deployment. When is full Kubernetes overkill?
For a small on-premise deployment k3s is usually the right choice, and full Kubernetes becomes overkill below genuine high-availability or scale needs. **k3s** is a certified, lightweight Kubernetes shipped as a single binary with a low memory footprint, using SQLite instead of etcd by default and bundling an ingress controller, while still exposing the same Kubernetes API — which makes it ideal for edge, IoT, and small clusters. Full Kubernetes runs multiple control-plane components plus etcd and carries higher CPU, memory, and operational overhead, which is justified only for large, multi-node, highly-available, autoscaling deployments with a rich ecosystem. When you have just a few nodes, modest workloads, and a small team, that control-plane overhead and complexity buy you nothing, so full k8s is overkill.

<a id="lo6-11"></a>
### Q11 · Docker Compose on one host vs go straight to Kubernetes

> **Full question:** Defend whether a small team should use Docker Compose on a single host or move straight to Kubernetes, considering complexity, scaling needs, and resilience.
A small team should start with Docker Compose unless it genuinely needs multi-node scaling and resilience right now. Compose on a single host is trivial to learn, uses one YAML file, and gives fast iteration, but it offers no high availability — if the host dies the app dies — and no autoscaling or cross-node self-healing. Kubernetes provides self-healing, horizontal scaling, multi-node resilience, and rolling updates, but at the cost of steep complexity and ongoing operational overhead. The practical decision rule is that a small app with predictable load on a single acceptable host should use Compose, while a system that needs uptime guarantees, scale-out, or many services spread across nodes justifies Kubernetes — and you should not pay the Kubernetes complexity tax before you actually need its benefits.

<a id="lo6-12"></a>
### Q12 · Vendor lock-in: Kubernetes (CNCF) vs OpenShift (Red Hat)

> **Full question:** Analyze vendor lock-in across Kubernetes (open source / CNCF) and OpenShift (Red Hat). How does the choice affect long-term flexibility?
Kubernetes minimizes lock-in because it is open and portable, whereas OpenShift trades some lock-in for an integrated, supported platform. Plain Kubernetes is vendor-neutral and CNCF-conformant, so standard objects port between clouds and distributions and give you maximum long-term flexibility. OpenShift runs standard Kubernetes underneath, but leaning on Routes, S2I, SCCs, and `oc`-specific workflows couples you to Red Hat, and migrating away means rebuilding those pieces, as in Q5. The effect on long-term flexibility comes down to how you build: staying on plain Kubernetes primitives keeps you easy to move, while relying on OpenShift-specific features delivers faster now but tighter coupling later, so the choice should follow how much you value portability versus an out-of-the-box supported platform.

<a id="lo6-13"></a>
### Q13 · Defend recommending OpenShift over vanilla k8s for an integrated, vendor-supported platform

> **Full question:** Defend a recommendation of OpenShift over vanilla Kubernetes for a company that wants an integrated, vendor-supported platform with built-in developer and operations tooling.
For a company that explicitly wants integrated developer and operations tooling with vendor support, I would recommend OpenShift as the stronger fit. It delivers CI/CD, an image registry, monitoring, logging, a developer console, and Routes out of the box, so the team avoids assembling a platform from separate parts. It is also secure by default through SCCs, RBAC, and OAuth, which removes much of the hardening work and helps with compliance, as in Q8. On top of that, Red Hat provides SLAs, patched releases, and a certified ecosystem, which lowers operational risk for an enterprise. The honest trade-off is the licence cost and some lock-in from Q12, but given the stated requirement for an integrated and supported platform, those costs are acceptable and the OpenShift recommendation is justified.

<a id="lo6-14"></a>
### Q14 · Storage provisioning: Kubernetes vs regular VM workloads

> **Full question:** Compare storage provisioning between Kubernetes and regular VM workloads.
Kubernetes storage is dynamic, declarative, and application-driven, whereas traditional VM storage is static and manually provisioned. With VMs an administrator pre-provisions a disk, attaches it, then formats and mounts it, which is manual, slow, and tied to a specific machine. In Kubernetes the application simply declares a **PVC**, and a **StorageClass** with a **CSI** driver provisions and attaches a volume on demand, even reattaching it when a pod moves to another node, with the volume's lifecycle governed declaratively through its reclaim policy. The contrast is that VM storage is imperative, human-driven, and host-bound, while Kubernetes storage is declarative, automated, and portable across nodes.

<a id="lo6-15"></a>
### Q15 · 2-engineer startup, ship fast & cheap — recommend a container solution

> **Full question:** A startup with two engineers must ship a containerized product quickly and cheaply. Recommend container solution and justify your choice on cost and operational overhead.
For a two-engineer startup that must ship quickly and cheaply, I would recommend Docker Compose on a single host, or a small managed PaaS, rather than Kubernetes. The reason is cost and operational overhead: one host and a single `compose.yaml` mean almost no cluster operations, so two engineers can spend their time on the product instead of running a control plane. Kubernetes would force them to learn and operate a cluster, handle upgrades, networking, and security, which is a tax a two-person team cannot afford at this stage. The sensible growth path is to move to a **managed Kubernetes** service such as EKS, GKE, or AKS only once they genuinely need high availability and scale, so that even then they avoid self-operating the control plane — matching the tool to the current stage of the company.

<a id="lo6-16"></a>
### Q16 · Docker vs Podman — architecture (daemon vs daemonless) and rootless security

> **Full question:** Compare Docker and Podman in terms of architecture (daemon vs daemonless) and rootless security. Why might a security-conscious team prefer Podman?
A security-conscious team should prefer **Podman**, because its daemonless and rootless-by-default design removes the biggest privileged attack surface that Docker carries. Docker relies on a central root daemon, `dockerd`, that every command talks to: it is a long-running, root-owned process and a single point of failure, so if a container escapes or the daemon is compromised the attacker lands directly on a root-level service. Podman has no such daemon — each command runs as its own short-lived process, and containers run rootless by default, mapping the container's root to an unprivileged host user through Linux user namespaces. This means a smaller attack surface, better isolation if a container is breached, and clean integration with systemd, which lets containers be managed as ordinary, auditable, non-root services. Docker can be run rootless too, but for Podman it is the default rather than an add-on, which is why teams that prioritize security tend to favour it.

<a id="lo6-17"></a>
### Q17 · Day-2 overhead: self-managed k8s vs managed k8s service

> **Full question:** Compare the day-2 operational overhead (cluster upgrades, scaling nodes, backups) of self-managed Kubernetes versus a managed Kubernetes service.
Managed Kubernetes drastically reduces day-2 operational overhead, while self-managed Kubernetes gives you control at the cost of carrying that burden yourself. Running it yourself means you own control-plane upgrades, etcd backups, node scaling and patching, high availability of the control plane, and security hardening, which is a heavy ongoing cost that requires real in-house expertise; the upside is full control, the ability to run anywhere including on-prem, and no per-cluster fee. A managed service such as EKS, GKE, or AKS hands the control plane to the provider, who handles upgrades, HA, and etcd backups and integrates autoscaling and cloud services, leaving you with far less toil but some cost and provider coupling and less low-level control. The verdict is that unless you have a strong platform team or specific control or on-prem requirements, managed Kubernetes is the pragmatic choice for day-2 operations.

<a id="lo6-18"></a>
### Q18 · Media-streaming company, spiky unpredictable global traffic — recommend a solution

> **Full question:** A media-streaming company expects spiky, unpredictable global traffic. Recommend a containerized solution. Elaborate your choice.
For a media-streaming company facing spiky, unpredictable, global traffic, I would recommend managed Kubernetes deployed across multiple regions with autoscaling and a CDN. Kubernetes fits because the Horizontal Pod Autoscaler scales pods with load, the Cluster Autoscaler adds and removes nodes to absorb spikes, and self-healing plus rolling updates keep the service resilient under churn. Going managed and multi-region delivers low global latency and lets the provider handle control-plane operations, and pairing this with a CDN offloads streaming bandwidth and caches content at the edge while LoadBalancer or Ingress distributes incoming traffic. The net result is elastic capacity for unpredictable spikes, genuine global reach, and minimal self-operated infrastructure, which plays directly to Kubernetes' strengths.

<a id="lo6-19"></a>
### Q19 · Outgrown Docker Swarm / single Compose host — migrate to Kubernetes?

> **Full question:** Defend whether a company that has outgrown Docker Swarm (or a single Compose host) should migrate to Kubernetes — weigh the migration effort against the long-term benefits.
Once a company has genuinely outgrown Docker Swarm or a single Compose host, it should migrate to Kubernetes, because the long-term benefits outweigh the one-time migration effort. The reason to migrate is that Swarm's ecosystem is stagnating while Kubernetes offers richer scaling through the HPA, self-healing, rolling updates and rollbacks, a vast ecosystem of operators, Helm, and service mesh, and the option of managed services. The migration effort is real and worth stating honestly: you must rewrite Compose or Swarm specifications as Kubernetes manifests or Helm charts, learn Kubernetes concepts, set up networking, storage, and CI, and train the team. The decision therefore hinges on whether you are actually hitting Swarm or Compose limits in scale, resilience, or ecosystem — if you are, the investment pays off, and if your current setup still meets your needs, you should not migrate merely for hype.

<a id="lo6-20"></a>
### Q20 · Would you choose Kubernetes for a microservices architecture? (orchestration, resilience, ecosystem)

> **Full question:** Would you choose Kubernetes as a solution for a microservices architecture, focus on its orchestration, resilience and ecosystem. Elaborate your standing with arguments.
Yes — Kubernetes is the natural fit for a microservices architecture. On orchestration it provides declarative deployments, per-service scaling, service discovery through Services and DNS, and independent rolling updates for each microservice. On resilience it self-heals by restarting and rescheduling failed pods, supports health probes, tolerates node faults across the cluster, and can roll back automatically. Its ecosystem is built around exactly this style of system, spanning Helm, operators, service meshes like Istio and Linkerd, and observability with Prometheus and Grafana, alongside GitOps workflows. The one caveat is the complexity overhead: for a handful of services on a single host Kubernetes can be overkill, as in Q11 and Q15, but for a real microservices estate it is the industry standard.

<a id="lo6-21"></a>
### Q21 · Would you choose Kubernetes for enterprise-grade apps? (scalability, community, features)

> **Full question:** Would you choose Kubernetes for enterprise-grade applications, focus on its scalability, community support and feature richness. Elaborate your standing with arguments.
Yes, I would choose Kubernetes for enterprise-grade applications. On scalability it offers horizontal pod and cluster autoscaling and is proven to thousands of nodes, handling large stateless and stateful workloads alike. On community support it is CNCF-graduated and the de-facto industry standard, with a huge contributor base, long-term viability, and broad vendor and managed-service backing. On feature richness it brings RBAC, network policies, secrets and config management, storage abstraction, rolling updates, operators, and extensibility through CRDs. The caveat is its operational complexity, which is why enterprises often pair it with managed services or with OpenShift, as in Q13, to gain support and reduce toil — but the standing remains a clear yes.

<a id="lo6-22"></a>
### Q22 · Would you choose OpenShift for a microservices architecture? (orchestration, resilience, ecosystem)

> **Full question:** Would you choose OpenShift as a solution for a microservices architecture, focus on its orchestration, resilience and ecosystem. Elaborate your standing with arguments.
Yes — OpenShift suits a microservices architecture well, particularly for an enterprise that wants an integrated platform. Because it is Kubernetes underneath, it inherits all of the orchestration and self-healing described for Kubernetes in Q20, with the addition of opinionated, secure defaults. Its ecosystem advantage is that CI/CD through Pipelines and GitOps, a service mesh, an image registry, monitoring, and Routes for exposing each service are integrated rather than assembled separately as they would be on vanilla Kubernetes. The trade-off is the licence cost and some Red Hat coupling from Q12, but for a security- and compliance-conscious organization running many microservices, that integration is worth it.

<a id="lo6-23"></a>
### Q23 · Would you choose OpenShift for enterprise-grade apps? (scalability, community, features)

> **Full question:** Would you choose OpenShift for enterprise-grade applications, focus on its scalability, community support and feature richness. Elaborate your standing with arguments.
Yes — OpenShift is arguably the strongest enterprise fit. On scalability it provides the full Kubernetes scalability of Q21 together with enterprise-tuned defaults. On community and support it is built on CNCF Kubernetes and additionally backed by Red Hat with commercial support, SLAs, a certified ecosystem, and patched releases, which is precisely what enterprises require. On feature richness it is secure by default through SCCs, RBAC, and OAuth, and ships integrated developer and operations tooling, S2I, Routes, GitOps, and compliance options. The trade-off is again cost and lock-in from Q12, but for regulated enterprises that need support and a complete feature set, OpenShift is the safe and feature-complete choice.

---

*End of LO6.*
