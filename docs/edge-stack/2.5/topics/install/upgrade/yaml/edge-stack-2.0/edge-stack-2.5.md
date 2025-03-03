import Alert from '@material-ui/lab/Alert';

# Upgrade $productName$ 2.0.5 to $productName$ $version$ (YAML)

<Alert severity="info">
  This guide covers migrating from $productName$ 2.0.5 to $productName$ $version$. If
  this is not your <b>exact</b> situation, see the <a href="../../../../migration-matrix">migration
  matrix</a>.
</Alert>

<Alert severity="warning">
  This guide is written for upgrading an installation made without using Helm.
  If you originally installed with Helm, see the <a href="../../../helm/edge-stack-2.0/edge-stack-2.5">Helm-based
  upgrade instructions</a>.
</Alert>

<Alert severity="warning">
  <b>Upgrading from $productName$ 2.0.5 to $productName$ $version$ typically requires downtime.</b>
  In some situations, Ambassador Labs Support may be able to assist with a zero-downtime migration;
  contact support with questions.
</Alert>

Migrating from $productName$ 2.0.5 to $productName$ $version$ is a three-step process:

1. **Install new CRDs.**

   Before installing $productName$ $version$ itself, you need to update the CRDs in
   your cluster. This is mandatory during any upgrade of $productName$.

   ```
   kubectl apply -f https://app.getambassador.io/yaml/edge-stack/$version$/aes-crds.yaml
   kubectl wait --timeout=90s --for=condition=available deployment emissary-apiext -n emissary-system
   ```

   <Alert severity="info">
     $productName$ $version$ includes a Deployment in the `emissary-system` namespace
     called <code>$productDeploymentName$-apiext</code>. This is the APIserver extension
     that supports converting $productName$ CRDs between <code>getambassador.io/v2</code>
     and <code>getambassador.io/v3alpha1</code>. This Deployment needs to be running at
     all times.
   </Alert>

   <Alert severity="warning">
     If the <code>$productDeploymentName$-apiext</code> Deployment's Pods all stop running,
     you will not be able to use <code>getambassador.io/v3alpha1</code> CRDs until restarting
     the <code>$productDeploymentName$-apiext</code> Deployment.
   </Alert>

2. **Delete $productName$ 2.0.5 Deployment.**

   <Alert severity="warning">
     Delete <b>only</b> the Deployment for $productName$ 2.0.5 in order to preserve all of
     your existing configuration.
   </Alert>

   Use `kubectl` to delete the Deployment for $productName$ 2.0.5. Typically, this will be found
   in the `ambassador` namespace.

   ```
   kubectl delete -n ambassador deployment edge-stack
   ```

3. **Install $productName$ $version$.**

   After installing the new CRDs, use Helm to install $productName$ $version$. This will install
   in the `$productNamespace$` namespace. If necessary for your installation (e.g. if you were
   running with `AMBASSADOR_SINGLE_NAMESPACE` set), you can download `aes.yaml` and edit as
   needed.

   ```
   kubectl apply -f https://app.getambassador.io/yaml/edge-stack/$version$/aes.yaml && \
   kubectl rollout status -n $productNamespace$ deployment/edge-stack -w
   ```
