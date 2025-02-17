## OPA/Gatekeeper

---

### Preventative Controls
Every change to a Kubernetes cluster is first stored in etcd, and only the API server modifies etcd; moreover, every change to a cluster goes through the API server. [Dynamic Admission Controllers](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/) are used to intercept API server requests, after they have been authenticated and authorized. `OPA/Gatekeeper` is used as a mutating and validating admission controller to mutate and validate API server requests before they result in changes to etcd and subsequent cluster changes. Using OPA/Gatekeeper policies, built using constraint templates and constraints, allows cluster operators and administrators to add `guardrails` to cluster operation, to prevent unwanted behaviors.

In this section of the workshop, attendees will install OPA/Gatekeeper to the cluster, and sample policies that act as preventative controls, preventing unwanted behaviors from happening within the cluster. The sample policies enforce the following controls:
- Container Security Context Elements in Deployment and Pod resources
- Host Network Access in Deployment and Pod resources
- Host Filesystem Access in Deployment and Pod resources

---

[OPA Gatekeeper: Policy and Governance for Kubernetes](https://kubernetes.io/blog/2019/08/06/opa-gatekeeper-policy-and-governance-for-kubernetes/)

[Gatekeeper GitHub Project](https://github.com/open-policy-agent/gatekeeper)

[Gatekeeper Policy Library](https://github.com/open-policy-agent/gatekeeper-library)

[Gatekeeper Installation Documentation](https://open-policy-agent.github.io/gatekeeper/website/docs/install/)

---

### Installation
```bash
helm repo add gatekeeper https://open-policy-agent.github.io/gatekeeper/charts
helm upgrade -i --create-namespace gatekeeper gatekeeper/gatekeeper --namespace gatekeeper \
--set release=v3.6.0
```

### Install Constraint Templates

```bash
kubectl apply -f constraint-templates/
```

### Install Constraints

```bash
kubectl apply -f constraints/
```

### Deinstallation

```bash
kubectl delete -f constraints/
kubectl delete -f constraint-templates/
helm uninstall gatekeeper
```

### Test

```bash
tests/test.sh
```
<sub><sup>(Note: Policy constraints and constraint-templates have been adapted from the Gatekeeper Library and the EKS Best Practices Guide.)</sup></sub>

---

### Deployment and Pod Policies
A combination of Gatekeeper constraint templates and constraints are used to apply policies. [OPA Rego](https://www.openpolicyagent.org/docs/latest/policy-language/) is written into the constraint template. Multiple constraints can be used with a constraint template. 

Both Deployment and Pod resource policies (constraint template and constraint combinations) have been added to this workshop. Applying policies to Deployment resources exposes immediate feedback to clients when they apply deployments, should the Deployment resource fail a policy. Pod policies are used to handle pods as well as handle resources that also create pods, such as Deployments and DaemonSets.

### Targeting Kubernetes Namespaces with Gatekeeper Policies
Each constraint is currently set to target the `test` namespace. To change the target namespace(s), edit the `spec.match.namespaces` array element, in each constraint, to include the target namespaces for each constraint.
```yaml
...
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
    namespaces: ["test"]
...
``` 

---

[EKS Best Practices Guides](https://aws.github.io/aws-eks-best-practices/)

[EKS Best Practices Guides - Policy Samples](https://github.com/aws/aws-eks-best-practices/tree/master/policies)
