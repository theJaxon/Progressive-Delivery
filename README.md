# Progressive-Delivery
Experimenting with Progressive delivery approach in Kubernetes by using Argo Rollouts controller

### Defining Progressive delivery
- Progressive delivery is a term that includes deployment strategies that try to avoid the pitfalls of all-or-nohting deployment strategies
- New versions being deployed don't replace existing versions but **`run in parallel`** for an amount of time receiving live production traffic and are evaluated in terms of correctness and performance before the rollout is considered successful (Before being fully `promoted`)
- The advantages of prgoressive delivery are
  - Avoiding downtime
  - Shorter time from idea to production
  - Limiting the blast radius

---

### Argo Rollouts
- A Kubernetes controller and set of CRDs which provide advanced deployment capabilities
- A `Rollout` is used instead of a deployment
- Both the Rollout and deployment defintions are almost identical with the excpetion of the added `strategy` section that indicates the type of strategy that we want to use
- There are many strategies supported by Argo Rollouts such as Blue Green and Canary ones

---

### References
- [Simple Kubernetes blue-green deployments](https://blog.nillsf.com/index.php/2019/11/10/simple-kubernetes-blue-green-deployments/) - Blog post explaining how to manually implement blue-green deployment strategy by changing `service selectors`
- [Progressive Delivery in Kubernetes - Carlos Sanchez, Adobe & Viktor Farcic, CloudBees](https://www.youtube.com/watch?v=Jf29YXu1Q48)