# Kubernetes

CREATED 2022/08/22

## Probes

**Readiness probes** - This probe will tell you when your app is ready to serve traffic. Kubernetes will ensure the readiness probe passes before allowing a service to send traffic to the pod. If the readiness probe fails, Kubernetes will not send the traffic to the pod until it passes.

**Liveness probes** - Liveness probes will let Kubernetes know whether your app is healthy. If your app is healthy, Kubernetes will not interfere with pod functioning, but if it is unhealthy, Kubernetes will destroy the pod and start a new one to replace it.

One case - The health check endpoint/path used in readiness check was set to a default health check provided by the Spring Boot Framework. And thus led readiness checks fail and caused an outage. See [Doordash Engineering Blog - How to Handle Kubernetes Health Checks](https://doordash.engineering/2022/08/09/how-to-handle-kubernetes-health-checks/) for details.

## Reference

[1] <https://thenewstack.io/kubernetes-health-checks-using-probes/>
