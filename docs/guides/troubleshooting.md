---
tags:
  - troubleshooting
  - faq
---

# Troubleshooting Guide

## Pod Stuck in `CrashLoopBackOff`

```bash
# Check pod logs
kubectl logs <pod-name> -n <namespace>

# Describe pod for events
kubectl describe pod <pod-name> -n <namespace>

# Common causes:
# - Missing environment variables
# - ConfigMap not mounted
# - Resource limits too low
```

## Node Not Ready

```bash
# List nodes and status
kubectl get nodes

# Describe the node
kubectl describe node <node-name>

# Check node conditions
kubectl get nodes -o json | jq '.items[].status.conditions'
```

## High Latency

1. Check application metrics in Grafana
2. Check database query performance
3. Verify pod resource limits
4. Check for network bottlenecks
5. Review recent deployments

## Common Commands

| Problem | Command |
|---------|---------|
| Pod crash | `kubectl logs --previous <pod>` |
| Port forward | `kubectl port-forward pod/<pod> 8080:80` |
| Exec into pod | `kubectl exec -it <pod> -- /bin/sh` |
| Get events | `kubectl get events --sort-by=.lastTimestamp` |
