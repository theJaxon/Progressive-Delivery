<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Blue Green with Argo Rollouts](#blue-green-with-argo-rollouts)
  - [Initial Deployment (Only 1 Revision)](#initial-deployment-only-1-revision)
    - [Service changes](#service-changes)
    - [Pod Changes](#pod-changes)
  - [Deploying a new version (Green one)](#deploying-a-new-version-green-one)
    - [Preview Service Changes](#preview-service-changes)
    - [Preview Pod Changes](#preview-pod-changes)
- [Post Promotion changes](#post-promotion-changes)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

### Blue Green with Argo Rollouts

#### Initial Deployment (Only 1 Revision)
- After deploying the Kubernetes resources including the Rollout via Kustomize, the controller introduces the following changes 

##### Service changes
- The controller changes the `selctor` of the service to include an additional key-value pair representing pod-template-hash
```yaml
  selector:
    ms: super-ms
    rollouts-pod-template-hash: 847d6bcdf7
```
- This hash is initially mapping to the blue pods as no changes were introduced so we only have 1 version of the application till this point

- The active and preview service section of the Rollout Strategy should confirm this 
```yaml
  blueGreen:
    activeSelector: 847d6bcdf7
    previewSelector: 847d6bcdf7
```

##### Pod Changes
- The pods for this Rollout also have their labels changed to include the `rollout-pod-template-hash`
```yaml
  labels:
    ms: super-ms
    rollouts-pod-template-hash: 847d6bcdf7
```
- Both the preview service and active service are pointing to the same pods, we can confirm it by checking the endpoints
```bash
# Get endpoints for super-ms-active service
k get ep -n argo-rollouts super-ms-active -ojson | jq .subsets[].addresses[].ip
"10.44.0.12"
"10.44.0.9"

# Get endpoints for super-ms-preview service
k get ep -n argo-rollouts super-ms-preview -ojson | jq .subsets[].addresses[].ip
"10.44.0.12"
"10.44.0.9"
```

```html
# Checking the deployed version 
# 192.168.100.10 is the address of the master machine from the Kontainerd project https://github.com/theJaxon/Kontainerd/blob/main/Vagrantfile#L10
# Port number is the NodePort for the ingress controller service
curl --proxy http://192.168.100.10:32703 argo-rollouts.io/blue-green
<html>
 <head>
  <title>Blue</title>
 </head>
 <body bgcolor="#11a7f7">
  <h1>V1</h1>
 </body>
</html>
```

![Revision-1](https://github.com/theJaxon/Progressive-Delivery/blob/main/etc/Blue-Green/Revision-1.jpg)

---

#### Deploying a new version (Green one)
- Initially when we deployed by using `kustomize build . | k apply -f -` we used the nginx image alongisde a configMap containing the word `v1` with blue background color
- We need to introduce a change for a new version to be detected and the change will be mounting the [`v2` configMap](https://github.com/theJaxon/Progressive-Delivery/blob/main/Blue-Green/green-conf/index.html) instead of the old one, by modifying volumeMounts in Rollout definition as follows 
```yaml
volumeMounts:
  - name: green-config
    mountPath: /usr/share/nginx/html
```
- This introduces revision 2 alongisde revision 1 and the controller did some changes to the labels and selectors again, this time to separate the preview service (that should point to the green version) from the active one pointing to the already running blue version
- A similar number of replicas is also now up and running (So we had 2 in the rollout and after the change we ended up with 4 replicas, 2 for the old, 2 for the new)
- Notice that even if everything is deployed fine and the new revision is ok no automatic promotion will happen as per specified in the rollout `autoPromotionEnabled: false`

##### Preview Service Changes 
- The preview service selector is now changed to point to the pod hash of the 2 newly created pods representing the green environment
```yaml
  selector:
    ms: super-ms
    rollouts-pod-template-hash: 67bfc8446
```

##### Preview Pod Changes 
- The label section of the pods now includes the `rollout-pod-template-hash` pointing to the 2 newly added pods 
```yaml
  labels:
    ms: super-ms
    rollouts-pod-template-hash: 67bfc8446
```
- At this point the endpoints for the preview and active services should be different as each is pointing to different pods because of the injected rollout-pod-template-hash

```bash
# Get endpoints for super-ms-active service
k get ep -n argo-rollouts super-ms-active -ojson | jq .subsets[].addresses[].ip
"10.44.0.12"
"10.44.0.9"

# Get endpoints for super-ms-preview service
k get ep -n argo-rollouts super-ms-preview -ojson | jq .subsets[].addresses[].ip
"10.44.0.17"
"10.44.0.18"
```

- Once we're confident that the introduced changes will not cause a problem we can `promote` the 2nd revision (Promotion will destroy the blue environment and the new version from that point should receive 100% of the traffic)

![Revision-2](https://github.com/theJaxon/Progressive-Delivery/blob/main/etc/Blue-Green/Revision-2.jpg)

---

#### Post Promotion changes 
- Both the active and preview services should now be pointing to the same pods so we should get the same endpoints 
```bash
# Get endpoints for super-ms-active service
k get ep -n argo-rollouts super-ms-active -ojson | jq .subsets[].addresses[].ip
"10.44.0.17"
"10.44.0.18"

# Get endpoints for super-ms-preview service
k get ep -n argo-rollouts super-ms-preview -ojson | jq .subsets[].addresses[].ip
"10.44.0.17"
"10.44.0.18"
```

```html
# Checking the deployed version 
# 192.168.100.10 is the address of the master machine from the Kontainerd project https://github.com/theJaxon/Kontainerd/blob/main/Vagrantfile#L10
# Port number is the NodePort for the ingress controller service
curl --proxy http://192.168.100.10:32703 argo-rollouts.io/blue-green
<html>
 <head>
  <title>Green</title>
 </head>
 <body bgcolor="#00FF00">
  <h1>V2</h1>
 </body>
</html>
```

![Post-Promotion](https://github.com/theJaxon/Progressive-Delivery/blob/main/etc/Blue-Green/Post-Promotion.jpg)