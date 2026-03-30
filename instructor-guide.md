# Instructor Guide: Scaling Strategies for AI Systems

## Module Overview
In this module, students will investigate methods for scaling AI systems horizontally and vertically. The focus is on learning how to balance efficiency, cost, and performance across distributed environments. Professionals will explore how Kubernetes primitives (like HPA and resource limits) manage the heavy compute demands of modern AI models while keeping cloud budgets in check.

## Learning Objectives
By the end of this session, students will be able to:
* Identify key factors influencing scalability in AI infrastructures.
* Configure horizontal and vertical scaling mechanisms in cloud environments.
* Assess the trade-offs between scalability, performance, and cost.

## Session Pacing Guide (120 Minutes total)

| Duration   | Segment                                 | Instructor Action                                                                                                                                                                                                            |
|:-----------|:----------------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **20 min** | **AI Scaling Complexities**             | Discuss the difference between scaling a Node.js app vs. a PyTorch model. Emphasize GPU constraints and memory limits.                                                                                                       |
| **40 min** | **HPA, VPA, and Limits**                | Live demo: Use Minikube to deploy a service. Enable the metrics server, apply an HPA, and use `scripts/load_generator.py` to send HTTP requests to the service from outside the cluster to watch pods scale out dynamically. |
| **60 min** | **Lab: Scaling and Optimization Setup** | Facilitate the lab where students create an optimized multi-stage Dockerfile, an HPA configuration, and a ResourceQuota. Ensure students grasp how smaller images improve HPA scale-up responsiveness.                       |

---

## Delivery Notes & Teaching Tips

### 1. Scaling vs. Sizing
* **Mid-Career Context:** Many professionals confuse "right-sizing" (finding the exact VM size needed) with "scaling" (elasticity). Emphasize that in Kubernetes, we right-size the *Pods* (Vertical / Resource Limits) and scale the *Deployments* (Horizontal).
* **Cost Reality:** Share a real-world horror story of an unrestricted auto-scaler spinning up dozens of expensive GPU nodes over a weekend due to a bug, illustrating why `maxReplicas` and namespace quotas are mandatory for reliability engineers.

### 2. Understanding Scale-Down Lag
* **Stabilization Window:** The scale-down stabilization window (often a 5-minute lag visible in the simulation data) is one of the most counter-intuitive HPA behaviors for students who expect elastic infrastructure to be perfectly reactive. 
* **Cost vs. Flapping:** Call out that scale-down intentionally lags to prevent system flapping. Be sure to explain that this lag carries a cost that engineers must consciously accept or tune.

### 3. Visualizing Autoscaling Metrics
Draw the feedback loop of the Horizontal Pod Autoscaler on a whiteboard:
1. `kubelet` collects metrics from the AI container.
2. Metrics Server aggregates the data.
3. HPA controller queries the Metrics Server.
4. If threshold is breached, HPA updates the Deployment's replica count.

---

## Common Student Issues & Solutions

| Issue                                 | Potential Cause                                                                             | Solution                                                                                                                                                  |
|:--------------------------------------|:--------------------------------------------------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------|
| **HPA shows `<unknown>/70%`**         | The Metrics Server is not installed or running in the cluster.                              | In Minikube, run `minikube addons enable metrics-server`. Wait 1-2 minutes for metrics to propagate.                                                      |
| **Pods are stuck in `Pending` state** | The requested resources (CPU/Memory) exceed what is available on the Minikube virtual node. | Teach students to inspect the failure using `kubectl describe pod [name]`. Lower the `requests` in the YAML file to fit the node capacity.                |
| **Multi-Stage Build fails (Lab)**     | Incorrectly copying artifacts between build stages in the multi-stage Dockerfile.           | Guide learners to verify their `COPY --from=builder` syntax and ensure dependencies like compiled AI libraries exist in the final minimal image layer.    |
| **OOMKilled Error**                   | The container exceeded its memory `limit`.                                                  | Explain that AI model inference often requires sudden spikes in memory. Adjust the `limit` higher, or implement chunked batch processing in the app code. |

---

## Debrief Questions
1. "In the context of scaling, how does utilizing a multi-stage Dockerfile to reduce image size directly improve the responsiveness of your Horizontal Pod Autoscaler?"
2. "If we deploy an AI model that takes 45 seconds to load its weights into memory, how does this affect our HPA strategy during a sudden traffic spike?"
3. "What is the danger of setting your Kubernetes Resource `Requests` equal to your `Limits` in a multi-tenant cluster?"

---

## Resources for the Instructor
* **Reference:** 
  * [Kubernetes Metrics Server](https://github.com/kubernetes-sigs/metrics-server)
  * [Architecting for Scale (AWS Machine Learning)](https://aws.amazon.com/machine-learning/)
* **AI Prompt Bank:**
    * *"Generate a Python script utilizing HTTP requests that we can use to trigger a Kubernetes HPA by sending high traffic to an API endpoint."*
    * *"Explain the difference between Kubernetes Horizontal Pod Autoscaler (HPA) and Cluster Autoscaler."*