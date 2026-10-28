# Lab 2 — hello-app on Kubernetes

Your copy of this repository is your submission. The lab sheet is on the course site; this file is where you record what you did.

What's here:

| File                 | What it is                                               |
| -------------------- | -------------------------------------------------------- |
| `src/`, `index.html` | a small React page that shows which Pod answered         |
| `nginx.conf`         | serves the page, and answers `/whoami` with the Pod name |
| `Dockerfile`         | builds the page, then serves it with nginx on port 80    |

You don't change any of these. You add `hello-app.yaml` and fill in the sections below.

## Before you push, check

- [ ] `hello-app.yaml` is in the root of this repository
- [ ] `screenshot.png` is in the root of this repository
- [ ] every section below is filled in

---

## 1. My image

Docker Hub image I deployed (for example `ravi123/hello-app:v1`):

```
ullas0422/hello-app:v1
```

## 2. Pods across nodes

Output of `kubectl get pods -o wide` after you applied `hello-app.yaml`:

```
NAME                         READY   STATUS    RESTARTS      AGE   IP            NODE              NOMINATED NODE   READINESS GATES
hello-app-54c9bcc954-6b6md   1/1     Running   0             6s    10.244.2.12   desktop-worker    <none>           <none>
hello-app-54c9bcc954-cjckm   1/1     Running   0             6s    10.244.1.12   desktop-worker2   <none>           <none>
hello-app-54c9bcc954-jfzn4   1/1     Running   0             6s    10.244.1.11   desktop-worker2   <none>           <none>
hello-app-54c9bcc954-k82j5   1/1     Running   0             6s    10.244.2.13   desktop-worker    <none>           <none>
who-556d9c4fcc-cffwc         1/1     Running   2 (78m ago)   13d   10.244.1.2    desktop-worker2   <none>           <none>
who-556d9c4fcc-gpppb         1/1     Running   2 (78m ago)   13d   10.244.2.2    desktop-worker    <none>           <none>
who-556d9c4fcc-kfgbv         1/1     Running   2 (78m ago)   13d   10.244.1.3    desktop-worker2   <none>           <none>
```

## 3. The Service and its endpoints

Output of `kubectl get service hello-app` and `kubectl get endpointslices -l kubernetes.io/service-name=hello-app`:

```
NAME        TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
hello-app   LoadBalancer   10.96.233.80   172.19.0.7    8080:31076/TCP   2m33s

NAME              ADDRESSTYPE   PORTS   ENDPOINTS                                         AGE
hello-app-qql6s   IPv4          80      10.244.2.12,10.244.1.11,10.244.1.12 + 1 more...   2m39s

```

## 4. Browser

Add `screenshot.png`: the page at `http://localhost:8080` with at least two Pods in the "Answers so far" table.

## 5. The four traps

For each trap, write what you saw and why it happened, in two or three lines of your own words.

**Trap 1 — the local image.**

The Docker image built on my local machine was not automatically available to Kubernetes through Docker Hub. I needed to tag the image with my Docker Hub username and push it to Docker Hub. I then used ullas0422/hello-app:v1 in my Deployment.

**Trap 2 — the selector typo.**
The Service selector must match the labels assigned to the Pods. When I used an incorrect selector, the Service could not find the intended Pods, so the EndpointSlice had no usable Pod endpoints. Changing the selector to app: hello-app fixed the mismatch.

**Trap 3 — the wrong targetPort.**
The application inside the container listens on port 80. When I changed targetPort to 8080, the Service forwarded traffic to the wrong container port and the application did not respond correctly. Restoring targetPort: 80 corrected the port configuration.

**Trap 4 — the EXTERNAL-IP address.**
The EXTERNAL-IP displayed by a Kubernetes LoadBalancer Service is not always the address I should open in my Windows browser. With Docker Desktop Kubernetes, I accessed the application through http://localhost:8080.

## 6. Your YAML vs the generated YAML

Name two fields in the `--dry-run=client -o yaml` output that were not in your file. Why does your file still work without them?

    Two fields in the generated YAML that may not be present in my manually written YAML are `creationTimestamp: null` and `resources: {}`. Kubernetes does not require every optional field to be explicitly specified. It can supply default values or operate without optional fields, so my manually written YAML can still work.
