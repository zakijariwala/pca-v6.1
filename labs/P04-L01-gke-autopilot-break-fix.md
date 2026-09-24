# LAB P04-L01: Break and fix a GKE Autopilot workload

## Goal
Create an Autopilot cluster, deploy an app with a Horizontal Pod Autoscaler, break it three ways (bad image, out of memory, unschedulable), fix each, then delete the cluster.

## What reading can't teach
What `ImagePullBackOff`, `OOMKilled`, and `Pending` look like in `kubectl describe`, and which line in the output names the cause.

## Cost ceiling
Under USD 2.00 for under 2 hours. GKE charges a cluster management fee by the hour; the GKE free tier credit covers one Autopilot or zonal cluster per billing account per month. Pods bill on their resource requests. [UNVERIFIED: check GKE pricing for your region.] Verified: 2026-09-24.

## Time
90 minutes, including teardown. Cluster creation takes several minutes.

## Prerequisites
- A sandbox project with a budget alert (LAB B01-L01).
- No other GKE cluster in the billing account, if you want the free tier credit to cover this one.
- Cloud Shell (it has `kubectl`).

## Steps (Cloud Shell)
Replace PROJECT_ID. Keep real IDs out of any file you commit.

```bash
export PROJECT_ID=PROJECT_ID
export REGION=us-central1
gcloud config set project "$PROJECT_ID"
gcloud services enable container.googleapis.com
```

1. Create the cluster and connect.
   ```bash
   gcloud container clusters create-auto p04 --region="$REGION"
   gcloud container clusters get-credentials p04 --region="$REGION"
   ```
2. Deploy the app with requests, and add an HPA.
   ```bash
   kubectl create deployment web --image=us-docker.pkg.dev/google-samples/containers/gke/hello-app:1.0
   kubectl set resources deployment web --requests=cpu=250m,memory=512Mi
   kubectl autoscale deployment web --cpu-percent=50 --min=1 --max=5
   kubectl get deploy,hpa,pods
   ```
3. Break 1: bad image tag.
   ```bash
   kubectl set image deployment/web hello-app=us-docker.pkg.dev/google-samples/containers/gke/hello-app:no-such-tag
   kubectl get pods
   kubectl describe pod POD_NAME | tail -20
   ```
   Fix it:
   ```bash
   kubectl rollout undo deployment/web
   ```
4. Break 2: out of memory. This Pod asks for more memory than its limit allows.
   ```bash
   cat <<'EOF' | kubectl apply -f -
   apiVersion: v1
   kind: Pod
   metadata:
     name: hog
   spec:
     containers:
     - name: hog
       image: polinux/stress
       command: ["stress"]
       args: ["--vm", "1", "--vm-bytes", "1500M", "--vm-hang", "1"]
       resources:
         requests: { memory: "1Gi", cpu: "250m" }
         limits:   { memory: "1Gi" }
   EOF
   kubectl get pod hog -w
   kubectl describe pod hog | grep -A5 "Last State"
   ```
   Fix: raise the limit above what the process uses, or lower `--vm-bytes`. Then delete the Pod.
   ```bash
   kubectl delete pod hog
   ```
5. Break 3: unschedulable. This Pod waits on a volume that can never bind.
   ```bash
   cat <<'EOF' | kubectl apply -f -
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: stuck
   spec:
     storageClassName: does-not-exist
     accessModes: ["ReadWriteOnce"]
     resources: { requests: { storage: 1Gi } }
   ---
   apiVersion: v1
   kind: Pod
   metadata:
     name: waiter
   spec:
     containers:
     - name: app
       image: us-docker.pkg.dev/google-samples/containers/gke/hello-app:1.0
       volumeMounts: [{ name: data, mountPath: /data }]
     volumes:
     - name: data
       persistentVolumeClaim: { claimName: stuck }
   EOF
   kubectl get pod waiter
   kubectl describe pod waiter | tail -10
   kubectl describe pvc stuck | tail -5
   ```
   Fix: point the claim at a storage class that exists (`kubectl get storageclass`). Then clean up.
   ```bash
   kubectl delete pod waiter && kubectl delete pvc stuck
   ```

## Expected output
- Step 2: `web` Deployment ready, HPA listed with target 50%.
- Step 3: Pod status `ErrImagePull`, then `ImagePullBackOff`; events name the missing tag.
- Step 4: Pod status `OOMKilled`, then `CrashLoopBackOff`; `Last State` shows exit code 137.
- Step 5: Pod stays `Pending`; events mention an unbound PersistentVolumeClaim, and the PVC event names the missing storage class.

Autopilot may adjust resource values in step 4 to meet its minimums or ratios. If the Pod doesn't OOM, raise `--vm-bytes`. [UNVERIFIED: current Autopilot minimums.]

## Teardown
Delete the cluster the same session. Its management fee bills by the hour.

```bash
gcloud container clusters delete p04 --region="$REGION" --quiet
gcloud container clusters list
```

## Fallback: delete the project
```bash
gcloud projects delete "$PROJECT_ID"
```
