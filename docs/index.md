# Pre-warmed K8s Clusters

Develop, test & demo your Kubernetes apps faster with pre-cached Docker images in pre-warmed Kubernetes clusters in microVMs.

For projects that test Kubernetes apps, standing up a Kubernetes cluster in CI can be painful, slow and flaky. Testfaster uses microVMs created by Ignite, which uses Firecracker, the open source technology from AWS which powers AWS Lambda, to:

* Pre-cache the fixed Docker images that your app uses, so that CI doesn't have to download them every time
* Start a Kubernetes cluster in a microVM for every test you want to run
* Pre-warm these Kubernetes clusters and keep a pool of them running for you so that your actual tests can run in seconds
* Your test suite just calls our API which either delivers a pre-warmed VM, or if the pool is empty, blocks until one is available
* Our API returns a kubeconfig file configured to point securely to the Kube cluster hosted on our infrastructure
* Caching network resources and pre-warming VMs can also significantly reduce the instance of flakiness in your test suite.
* Integrates with Travis.CI, GitLab, and any other CI system you are using!

Try it now by getting a [free access token](https://testfaster.ci/access_token).

## Experience the joy of integration tests that run in seconds

* Speed up delivery of software projects by landing code faster
* Improve correctness by testing more, reduce the bugs that reach prod
* Liberate your developers from context-switching

### <em>"Testfaster.ci is my dev team's secret weapon"</em>
<div style="margin-left: 2em;"><em>-- Anonymous</em></div>
