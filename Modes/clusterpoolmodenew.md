
# Cilium IPAM Mode Evaluation on AWS EKS

## Configuration Timeline

### Phase 1: Cluster Pool with Native Routing
```yaml
ipam:
  mode: cluster-pool
routingMode: native 
ipv4NativeRoutingCIDR: 10.8.0.0/16
autoDirectNodeRoutes: true
```

**Results:**
- ✅ Successful installation
- `cilium status` showed:
  ```
  IPv4: 4/254 allocated in 10.0.1.0/24
  ```
- ❌ Cluster health: 1/14 nodes reachable
- ❌ Pod scheduling: Only ~40/160 pods scheduled (Pending state)
- ❗ Resource utilization showed available capacity

### Phase 2: Disabled Native Routing
```diff
- ipv4NativeRoutingCIDR: 10.8.0.0/16
- autoDirectNodeRoutes: true
+ tunnel: vxlan
```

**Results:**
- ✅ Cluster health improved to 14/14 reachable
- ❌ Pod scheduling unchanged (~40/254)
- ❌ New issues:
  - perf-app-service failures
  - cert-manager webhook timeouts
  - DNS resolution failures (timeout to 10.100.0.10 - kube-dns)

### Phase 3: ENI Mode with Prefix Delegation
```yaml
ipam:
  mode: eni
routingMode: native
enableIPv4PrefixDelegation: true
```

## Key Observations

1. **Cluster Pool Limitations:**
   - Native routing caused node reachability issues
   - Pod scheduling capped at ~40 despite CIDR space for 254

2. **Connectivity Problems:**
   - DNS resolution failures impacted critical services
   - cert-manager webhooks failed due to network timeouts

3. **ENI Mode Benefits:**
   - Native AWS networking integration
   - Better scaling with prefix delegation
   - Improved pod density per node

## Recommended Actions

1. **Verify ENI Configuration:**
   ```bash
   kubectl exec -n kube-system <cilium-pod> -- \
     cilium status | grep -A5 "IPAM"
   ```

2. **Check AWS Limits:**
   ```bash
   aws ec2 describe-instance-types \
     --query "InstanceTypes[?InstanceType=='<your-instance-type>'].NetworkInfo"
   ```

3. **Monitor Pod Scheduling:**
   ```bash
   watch 'kubectl get pods -o wide | grep Pending'
   ```

## Lessons Learned
- Native routing requires precise VPC configuration
- ENI mode provides better AWS integration
- Pod scheduling limits may stem from multiple factors:
  - IPAM constraints
  - AWS ENI limits
  - Underlying network policies
```
