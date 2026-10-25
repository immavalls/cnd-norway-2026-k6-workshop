# Distributed load testing with the k6 Operator

_**Need help?** Raise your hand and we'll come help._

In this exercise, you'll learn how to:

- Run a k6 test as a Kubernetes custom resource
- See how the k6 Operator turns one `TestRun` into multiple Kubernetes Jobs
- Inspect the output from distributed k6 runner processes

The system under test does not technically need to run in Kubernetes. But in this lab, we'll get you to deploy QuickPizza into your own cluster, so your test results are yours and yours alone.

Run all commands from the repository root.

> **Windows (PowerShell) users:** the multi-line commands below use `\` to continue a line. In PowerShell, either put the command on one line or end each line with a backtick (`` ` ``) instead. In the clean-up commands, drop `2>/dev/null || true`.

If the workshop Docker Compose stack is still running and your laptop has limited memory, stop it and remove its volumes before starting kind:

```bash
docker compose down -v
```

## Install kind and kubectl

This exercise uses [kind](https://kind.sigs.k8s.io/) (Kubernetes in Docker) to create a local cluster, and [kubectl](https://kubernetes.io/docs/reference/kubectl/) to interact with it. If you don't have them installed, follow the installation instructions:

- [Install kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)
- [Install kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl)

Then verify that both work:

```bash
kind version
kubectl version --client
```

## Create the cluster

Create a one-node kind cluster and confirm that `kubectl` is using it. The first time, this downloads the node image and can take a few minutes:

```bash
kind create cluster --name k6-workshop
kubectl config current-context
```

The context should be `kind-k6-workshop`.

Install the pinned [k6 Operator v1.7.0](https://github.com/grafana/k6-operator/releases/tag/v1.7.0) bundle, and wait for its controller to become available:

```bash
kubectl apply --server-side -f https://raw.githubusercontent.com/grafana/k6-operator/v1.7.0/bundle.yaml
kubectl wait --for=condition=available deployment/k6-operator-controller-manager \
  --namespace k6-operator-system --timeout=120s
```

The bundle installs the k6 Custom Resource Definitions, the operator controller, and the permissions it needs to create Jobs and Pods.

Check that the output is this:

```bash
namespace/k6-operator-system serverside-applied
customresourcedefinition.apiextensions.k8s.io/privateloadzones.k6.io serverside-applied
customresourcedefinition.apiextensions.k8s.io/testruns.k6.io serverside-applied
serviceaccount/k6-operator-controller serverside-applied
role.rbac.authorization.k8s.io/k6-operator-leader-election-role serverside-applied
clusterrole.rbac.authorization.k8s.io/k6-operator-manager-role serverside-applied
clusterrole.rbac.authorization.k8s.io/k6-operator-metrics-auth-role serverside-applied
clusterrole.rbac.authorization.k8s.io/k6-operator-metrics-reader serverside-applied
rolebinding.rbac.authorization.k8s.io/k6-operator-leader-election-rolebinding serverside-applied
clusterrolebinding.rbac.authorization.k8s.io/k6-operator-manager-rolebinding serverside-applied
clusterrolebinding.rbac.authorization.k8s.io/k6-operator-metrics-auth-rolebinding serverside-applied
service/k6-operator-controller-manager-metrics-service serverside-applied
deployment.apps/k6-operator-controller-manager serverside-applied
deployment.apps/k6-operator-controller-manager condition met
```

## Deploy QuickPizza

Deploy QuickPizza and wait for its readiness probe to pass:

```bash
kubectl apply -f 6.operator/quickpizza.yaml
kubectl rollout status deployment/quickpizza --timeout=120s
kubectl get deployment,service quickpizza
```

The output should look similar to:

```bash
deployment.apps/quickpizza created
service/quickpizza created
Waiting for deployment "quickpizza" rollout to finish: 0 of 1 updated replicas are available...
deployment "quickpizza" successfully rolled out
NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/quickpizza   1/1     1            1           3s

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/quickpizza   ClusterIP   10.96.85.180   <none>        3333/TCP   3s
```

The `quickpizza` ClusterIP Service exposes port 3333. Workloads in the same namespace can reach it at `http://quickpizza:3333`.

### Check the deployment

To check that QuickPizza is running in your cluster, forward its port to your machine:

```bash
kubectl port-forward svc/quickpizza 3333:3333
```

Open [http://localhost:3333](http://localhost:3333) in your browser. You should see the QuickPizza page. Then press `Ctrl+C` to stop the port forward. You don't need it for the rest of the exercise, because the k6 runners reach QuickPizza inside the cluster.

## Create and run the distributed test

A ConfigMap makes the local test script available inside the runner Pods. Create it from the supplied script:

```bash
kubectl create configmap quickpizza-test \
  --from-file=k6-test.js=6.operator/k6-test.js
```

Check that the output is this:

```bash
configmap/quickpizza-test created
```

Review [`testrun.yaml`](./testrun.yaml). Its main fields are:

- `script.configMap`: mounts the `k6-test.js` ConfigMap entry as the test script.
- `parallelism: 2`: splits one logical k6 test run into two equal runners.
- `separate: false`: allows both runner Pods to run on the same node.
- `runner.env`: gives the runners QuickPizza's in-cluster URL.

Apply the `TestRun`:

```bash
kubectl apply -f 6.operator/testrun.yaml
```

Check that the output is this:

```bash
testrun.k6.io/quickpizza-test created
```

Right after applying, look at the resources created by the operator. The test lasts about a minute, so run this command a few times while it runs:

```bash
kubectl get testruns,jobs,pods
```

Alternatively, in a second terminal, add `--watch` to follow the changes live (press `Ctrl+C` to stop):

```bash
kubectl get testruns,jobs,pods --watch
```

The output should look similar to:

```bash
NAME                            STAGE     AGE   TESTRUNID
testrun.k6.io/quickpizza-test   started   56s

NAME                                    STATUS     COMPLETIONS   DURATION   AGE
job.batch/quickpizza-test-1             Running    0/1           53s        53s
job.batch/quickpizza-test-2             Running    0/1           53s        53s
job.batch/quickpizza-test-initializer   Complete   1/1           4s         56s
job.batch/quickpizza-test-starter       Complete   1/1           3s         50s

NAME                                    READY   STATUS      RESTARTS   AGE
pod/quickpizza-699984dfb6-mfhxn         1/1     Running     0          4m52s
pod/quickpizza-test-1-l24r4             1/1     Running     0          53s
pod/quickpizza-test-2-r48nk             1/1     Running     0          53s
pod/quickpizza-test-initializer-85cr4   0/1     Completed   0          56s
pod/quickpizza-test-starter-hrbkv       0/1     Completed   0          50s
```

You should see four Jobs over the lifetime of the test: one initializer, two k6 runners, and one starter:

- The initializer validates and prepares the script.
- Each runner owns one execution segment.
- The starter coordinates their common start.

Since you're running on _kind_, your cluster will have only one node. The two runner Pods that we've scheduled here show how k6 test execution can be split and orchestrated by the operator.

Wait for the TestRun to finish, then see what happened:

```bash
kubectl wait --for=jsonpath='{.status.stage}'=finished \
  testrun/quickpizza-test --timeout=180s
kubectl get testrun quickpizza-test
kubectl get jobs
```

The output should look similar to:

```bash
testrun.k6.io/quickpizza-test condition met
NAME              STAGE      AGE     TESTRUNID
quickpizza-test   finished   2m18s
NAME                          STATUS     COMPLETIONS   DURATION   AGE
quickpizza-test-1             Complete   1/1           67s        2m15s
quickpizza-test-2             Complete   1/1           67s        2m15s
quickpizza-test-initializer   Complete   1/1           4s         2m18s
quickpizza-test-starter       Complete   1/1           3s         2m12s
```

## Compare the runner output

Read the summary from each runner:

```bash
kubectl logs job/quickpizza-test-1
kubectl logs job/quickpizza-test-2
```

The output should look similar to the familiar k6 end-of-test summary:

```bash
  █ TOTAL RESULTS

    checks_total.......: 121     2.013378/s
    checks_succeeded...: 100.00% 121 out of 121
    checks_failed......: 0.00%   0 out of 121

    ✓ status is 200
...
```

In the summaries:

- Both runners should show successful iterations and a 100% `status is 200` check.
- The script configures a total arrival rate of 4 iterations per second for 1 minute.
- The operator has divided that rate across the two execution runners; it does not run 4 iterations per second on each runner.

If you add the `iterations` values from both summaries, you should get approximately 240, albeit with some possible variation.

## Optional: compare one runner

Delete the completed `TestRun` before reusing its name. Otherwise its completed Jobs can collide with the next run:

```bash
kubectl delete -f 6.operator/testrun.yaml --ignore-not-found
```

Change `parallelism` in [`testrun.yaml`](./testrun.yaml) from `2` to `1`, apply it again, and watch the resources:

```bash
kubectl apply -f 6.operator/testrun.yaml
kubectl get testruns,jobs,pods --watch
```

This run creates one runner Job, while the test still targets approximately 4 iterations per second and 240 total iterations. Delete the `TestRun` when you finish. Restore `parallelism: 2` if you want to keep the supplied file unchanged.

## Troubleshooting

**If you re-apply the TestRun onto the cluster, it does not start a new test**

A completed `TestRun` is not restarted by `kubectl apply`. Delete the TestRun before applying it again:

```bash
kubectl delete testrun quickpizza-test --ignore-not-found
kubectl apply -f 6.operator/testrun.yaml
```

## Clean up

Remove only the lab workloads and operator, while keeping the kind cluster:

```bash
kubectl delete testrun quickpizza-test --ignore-not-found 2>/dev/null || true
kubectl delete configmap quickpizza-test --ignore-not-found
kubectl delete -f 6.operator/quickpizza.yaml --ignore-not-found
kubectl delete -f https://raw.githubusercontent.com/grafana/k6-operator/v1.7.0/bundle.yaml \
  --ignore-not-found
```

Or remove the entire cluster, including all lab resources:

```bash
kind delete cluster --name k6-workshop
```

## Related Resources

- [k6 Operator documentation](https://grafana.com/docs/k6/latest/set-up/set-up-distributed-k6/)
- [Run k6 scripts with the TestRun CRD](https://grafana.com/docs/k6/latest/set-up/set-up-distributed-k6/usage/executing-k6-scripts-with-testrun-crd/)
- [Configure the TestRun CRD](https://grafana.com/docs/k6/latest/set-up/set-up-distributed-k6/usage/configure-testrun-crd/)
- [kind documentation](https://kind.sigs.k8s.io/)

---

[← Previous exercise](../5.infrastructure-testing/) · [Workshop homepage](https://github.com/immavalls/cnd-norway-2026-k6-workshop)
