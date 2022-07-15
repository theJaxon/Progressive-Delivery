### Canary with Argo Rollouts
#### Initial Deployment
- The Rollout resource specifies initially `5` replicas and in it the **setWeight** is set to 20% meaning that it will affect only 1 out of the five pods
- After a change is made the rollout will pause (No automatic promotion and no more than 1 pod out of 5 will be replaced)

```yaml
  strategy:
    canary: 
      steps:
      - setWeight: 20
      - pause: {}
```

- At this point V1 is receiving all the traffic, we can check the five pods and the response we're getting
```bash
for i in {1..5}; do curl --proxy http://192.160:32703 argo-rollouts.io/canary && echo -e "\n"; done

<!DOCTYPE html>
<html>
<body> <h3>VERSION 1</h3> </body>
</html>

<!DOCTYPE html>
<html>
<body> <h3>VERSION 1</h3> </body>
</html>

<!DOCTYPE html>
<html>
<body> <h3>VERSION 1</h3> </body>
</html>

<!DOCTYPE html>
<html>
<body> <h3>VERSION 1</h3> </body>
</html>

<!DOCTYPE html>
<html>
<body> <h3>VERSION 1</h3> </body>
</html>
```

![Revision-1](https://github.com/theJaxon/Progressive-Delivery/blob/main/etc/Canary/Revision-1.jpg)

---

#### Introducing change to the Rollout
- Similar change will be done to the Rollout resource where we will use V2 of the ConfigMap instead of V1
- Argo Rollouts now creates a new ReplicaSet with 1 Replica for the new Version as specified and the old ReplicaSet now has 4 desired replicas instead of 5

```yaml
# Mount V2 CM instead of V1
volumeMounts:
- mountPath: /usr/share/nginx/html
  name: v2-config
```
- It's expected now that when we curl five times we get V2 one time and V1 4 times, however that's not always the case as pods are picked randomly

```bash
# Here V2 can be seen twice because each time a random pod is selected
for i in {1..5}; do curl --proxy http://192.168.100.10:32703 argo-rollouts.io/canary && echo -e "\n"; done
<!DOCTYPE html>
<html>
<body> <h3>VERSION 1</h3> </body>
</html>

<!DOCTYPE html>
<html>
<body> <h3>VERSION 2</h3> </body>
</html>

<!DOCTYPE html>
<html>
<body> <h3>VERSION 1</h3> </body>
</html>

<!DOCTYPE html>
<html>
<body> <h3>VERSION 1</h3> </body>
</html>

<!DOCTYPE html>
<html>
<body> <h3>VERSION 2</h3> </body>
</html>
```

![Revision-2](https://github.com/theJaxon/Progressive-Delivery/blob/main/etc/Canary/Revision-2.jpg)