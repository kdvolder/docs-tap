# Troubleshooting Cloud Native Runtimes

This topic tells you how to troubleshoot Cloud Native Runtimes, commonly known as CNRs, installation or configuration.

## <a id='aws-connection-failure'></a> Cannot connect to app on AWS

### Symptom

On AWS, you see the following error when connecting to your app:

```console
curl: (6) Could not resolve host: a***********************7.us-west-2.elb.amazonaws.com
```

### Solution

Try connecting to your app again after 5 minutes. The AWS LoadBalancer name resolution takes
several minutes to propagate.

## <a id='imgpkg-pull-overwriting'></a> Pulling an image with imgpkg overwrites the cloud-native-runtimes directory

### Symptom

When relocating an image to a private registry and later pulling that image with
`imgpkg pull --lock LOCK-OUTPUT -o ./cloud-native-runtimes`, the contents of the cloud-native-runtimes are overwritten.

### Solution

Upgrade the imgpkg version to v0.13.0 or later.

## <a id='reconcile-fails'></a> Installation fails to reconcile app/cloud-native-runtimes

### Symptom

When installing Cloud Native Runtimes, you see one of the following errors:

```console
11:41:16AM: ongoing: reconcile app/cloud-native-runtimes (kappctrl.k14s.io/v1alpha1) namespace: cloud-native-runtime
11:41:16AM:  ^ Waiting for generation 1 to be observed
kapp: Error: Timed out waiting after 15m0s
```

Or,

```console
3:15:34PM:  ^ Reconciling
3:16:09PM: fail: reconcile app/cloud-native-runtimes (kappctrl.k14s.io/v1alpha1) namespace: cloud-native-runtimes
3:16:09PM:  ^ Reconcile failed:  (message: Deploying: Error (see .status.usefulErrorMessage for details))

kapp: Error: waiting on reconcile app/cloud-native-runtimes (kappctrl.k14s.io/v1alpha1) namespace: cloud-native-runtimes:
  Finished unsuccessfully (Reconcile failed:  (message: Deploying: Error (see .status.usefulErrorMessage for details)))
```

### Explanation

The `cloud-native-runtimes` deployment app installs the subcomponents of Cloud Native Runtimes.
Error messages about reconciling indicate that one or more subcomponents have failed to install.

### Solution

Use the following procedure to examine logs:

1. Get the logs from the `cloud-native-runtimes` app by running:

    ```console
    kubectl get app/cloud-native-runtimes -n cloud-native-runtimes -o jsonpath="{.status.deploy.stdout}"
    ```

    > **Note** If the command does not return log entries,  kapp-controller is not installed or is not running correctly.

2. Review the output for sub component deployments that have failed or are still ongoing.
   See the following examples for suggestions on resolving common problems.

#### Example 1: The Cloud Provider does not support the creation of Service type LoadBalancer

Follow these steps to identify and resolve the problem of the cloud provider not supporting services of type `LoadBalancer`:

1. Search the log output for `Load balancer`, for example by running:

    ```console
    kubectl -n cloud-native-runtimes get app cloud-native-runtimes -ojsonpath="{.status.deploy.stdout}" | grep "Load balancer" -C 1
    ```

2. If the output looks similar to the following,
   ensure that your cloud provider supports services of type `LoadBalancer`.
   For more information, see [Prerequisites](../../tap-gui/accessing-tap-gui.hbs.md).

    ```console
    6:30:22PM: ongoing: reconcile service/envoy (v1) namespace: CONTOUR-NS
    6:30:22PM:  ^ Load balancer ingress is empty
    6:30:29PM: ---- waiting on 1 changes [322/323 done] ----
    ```

    Where `CONTOUR-NS` is the namespace where Contour is installed on your cluster. If Cloud Native Runtimes was installed as part of a Tanzu Application Profile, this value is likely `tanzu-system-ingress`.

#### Example 2: The webhook deployment failed

Follow these steps to identify and resolve the problem of the `webhook` deployment failing in the `knative-serving` namespace:

1. Review the logs for output similar to the following:

    ```console
    10:51:58PM: ok: reconcile customresourcedefinition/httpproxies.projectcontour.io (apiextensions.k8s.io/v1) cluster
    10:51:58PM: fail: reconcile deployment/webhook (apps/v1) namespace: knative-serving
    10:51:58PM:  ^ Deployment is not progressing: ProgressDeadlineExceeded (message: ReplicaSet "webhook-6f5d979b7d" has timed out progressing.)
    ```

1. Run `kubectl get pods` to find the name of the pod:

    ```console
    kubectl get pods --show-labels -n NAMESPACE
    ```

    Where `NAMESPACE` is the namespace associated with the reconcile error, for example, `knative-serving`.

    For example,

    ```console
    $ kubectl get pods --show-labels -n knative-serving
    NAME                       READY   STATUS    RESTARTS   AGE   LABELS
    webhook-6f5d979b7d-cxr9k   0/1     Pending   0          44h   app=webhook,kapp.k14s.io/app=1626302357703846007,kapp.k14s.io/association=v1.9621e0a793b4e925077dd557acedbcfe,pod-template-hash=6f5d979b7d,role=webhook
    ```

2. Run `kubectl logs` and `kubectl describe`:

    ```console
    kubectl logs PODNAME -n NAMESPACE
    kubectl describe pod PODNAME -n NAMESPACE
    ```

    Where:

    - `PODNAME` is found in the output of step 3. For example, `webhook-6f5d979b7d-cxr9k`.
    - `NAMESPACE` is the namespace associated with the reconcile error, for example, `knative-serving`.

    For example:

    ```console
    $ kubectl logs webhook-6f5d979b7d-cxr9k -n knative-serving

    $ kubectl describe pod webhook-6f5d979b7d-cxr9k  -n knative-serving
    Events:
    Type     Reason            Age                 From               Message
    ----     ------            ----                ----               -------
    Warning  FailedScheduling  80s (x14 over 14m)  default-scheduler  0/1 nodes are available: 1 Insufficient cpu.
    ```

3. Review the output from the `kubectl logs` and `kubectl describe` commands and take further action.

    For this example of the webhook deployment, the output indicates that the scheduler does not have enough CPU to run the pod.
    In this case, the solution is to add nodes or CPU cores to the cluster.
    If you are using Tanzu Mission Control (TMC), increase the number of workers in the node pool
    to three or more through the TMC UI.
    See [Edit a Node Pool](https://docs.vmware.com/en/VMware-Tanzu-Mission-Control/services/tanzumc-using/GUID-53D4E904-3FFE-464A-8814-13942E03232A.html),
    in the TMC documentation.

## <a id='aws-connection-failure'></a> Cloud Native Runtimes Installation Fails with Existing Contour Installation

### Symptom

You see the following error message when you run the install script:

  ```console
  Could not proceed with installation. Refer to Cloud Native Runtimes documentation for details on how to utilize an existing Contour installation. Another app owns the custom resource definitions listed below.
  ```

### Solution

Follow the procedure in
[Install Cloud Native Runtimes on a Cluster with Your Existing Contour Instances](./contour.hbs.md#install-with-existing-contour)
to resolve the issue.

## <a id='invalid-httpproxy'></a> Knative Service Fails to Come up Due to Invalid HTTPPRoxy

### Symptom

When creating a Knative Service, it does not reach ready status. The corresponding Route resource has the status  `Ready=Unknown` with `Reason=EndpointsNotReady`. When you inspect the logs for the `net-contour-controller`, you see an error like this:

```json
{"severity":"ERROR","timestamp":"2022-12-08T16:27:08.320604183Z","logger":"net-contour-controller","caller":"ingress/reconciler.go:313","message":"Returned an error","commit":"041f9e3","knative.dev/controller":"knative.dev.net-contour.pkg.reconciler.contour.Reconciler","knative.dev/kind":"networking.internal.knative.dev.Ingress","knative.dev/traceid":"9d615387-f552-449c-a8cd-04c69dd1849e","knative.dev/key":"cody/foo-java","targetMethod":"ReconcileKind","error":"HTTPProxy.projectcontour.io \"foo-java-contour-5f549ae3e6f584a5f33d069a0650c0d8foo-java.cody.\" is invalid: metadata.name: Invalid value: \"foo-java-contour-5f549ae3e6f584a5f33d069a0650c0d8foo-java.cody.\": a lowercase RFC 1123 subdomain must consist of lower case alphanumeric characters, '-' or '.', and must start and end with an alphanumeric character (e.g. 'example.com', regex used for validation is '[a-z0-9]([-a-z0-9]*[a-z0-9])?(\\.[a-z0-9]([-a-z0-9]*[a-z0-9])?)*')","stacktrace":"knative.dev/networking/pkg/client/injection/reconciler/networking/v1alpha1/ingress.(*reconcilerImpl).Reconcile\n\tknative.dev/networking@v0.0.0-20221012062251-58f3e6239b4f/pkg/client/injection/reconciler/networking/v1alpha1/ingress/reconciler.go:313\nknative.dev/pkg/controller.(*Impl).processNextWorkItem\n\tknative.dev/pkg@v0.0.0-20221011175852-714b7630a836/controller/controller.go:542\nknative.dev/pkg/controller.(*Impl).RunContext.func3\n\tknative.dev/pkg@v0.0.0-20221011175852-714b7630a836/controller/controller.go:491"}
```

### Solution

Due to a [known upstream Knative issue](https://github.com/knative/pkg/issues/2659), certain combinations of Name, Namespace, and Domain yield invalid names for HTTPProxy resources due to the way the name is hashed and trimmed to fit the size requirement. It can have non-alphanumeric characters at the end of the name.

Resolving this is unique to each Knative service. It is likely to involve renaming your app to be shorter so that after the hash and trim procedure, the name is cut to end on an alphanumeric character.

For example, `foo-java.cody.iterate.tanzu-azure-lab.winterfell.fun` is hashed and trimmed into `foo-java-contour-5f549ae3e6f584a5f33d069a0650c0d8foo-java.cody.`, leaving an invalid `.` at the end.

However, changing the app name to `foo-jav` causes `foo-jav-contour-<some different hash>foo-jav.cody.it`, which is a valid name.

## <a id='certificate-not-ready-kcert'></a> When using auto-tls, Knative Service Fails with `CertificateNotReady`

### Symptom

When creating a Knative Service, it does not reach ready status. The Knative Service has the status  `CertificateNotReady`. 
When you verify the status of the `kcert` resource that belongs to the Knative Service you see a message like this:

```console
kubectl -n your-namespace get kcert route-76e387a2-cc35-4580-b2f1-bf7561371891 -ojsonpath='{.status}'
```

Output:

```json
{
  "conditions":[
  {
    "lastTransitionTime":"2023-06-05T11:26:53Z",
    "message":"error creating Certmanager Certificate: cannot create valid length CommonName: (where-for-dinner.medium.longevityaks253.tapalong.cloudfocused.in) still longer than 63 characters, cannot shorten",
    "reason":"CommonName Too Long",
    "status":"False",
    "type":"Ready"
  }],
  "observedGeneration":1}
```

### Solution

Due to a restriction imposed by cert-manager, CNs cannot be longer than 64 bytes (see [here](https://github.com/cert-manager/cert-manager/issues/1462)). For Knative using cert-manager, this means that the FQDN for a Knative Service (usually comprised of `<ksvc name>.<namespace>.<domain>`, but configurable via `domain_template` in CNRs) must not exceed 64 bytes.

[There is an open issue in Knative Serving community that aims to solve this](https://github.com/knative-sandbox/net-certmanager/issues/550).

To avoid this, deactivate TLS. See [CNRs docs on disabling auto tls](./auto-tls/tls-guides-deactivate-autotls.hbs.md).

If you want to continue using TLS, there are a few ways to resolve this on your own, though each has its own risks and limits.

#### Option 1: Change the `domain_template`

Changing the `domain_template` alters how Knative will create FQDNs for Knative Services. See [configuring External DNS](./external_dns.hbs.md#configure-knative-service-domain-template).

You can use this option to shorten the template, either by shortening one of the fields:

```console
\{{.Name}}.\{{slice .Namespace 0 3}}.\{{.Domain}}
```

> **Note:** Knative was not designed with shortening the name or namespace in mind. Due to a quirk in Knative's domain template validation, you can only slice up to a max of 3 characters.

Or by removing a field altogether:

```console
\{{.Name}}.\{{.Domain}}
```

> **Caution** Removing the namespace from the `domain_template` makes it possible for Knative to create non-unique FQDNs for Knative Services across different namespaces. It requires manual care in naming Knative Services to ensure that FQDNs remain unique.

#### Option 2: Shorten the names of Knative Services or Namespaces

You can also shorten the names of your Knative Services or Namespaces, if you have that ability. This requires some manual calculation to ensure that the shortened Name, Namespace, and domain,including `.`s, come out to less that 64 bytes.
