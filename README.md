# Mixer Out of Process LogEntry Adapter Walkthrough

This is an adaptation and expansion of the [Mixer Out of Process Adapter (MOPA) Walkthrough](https://github.com/istio/istio/wiki/Mixer-Out-of-Process-Adapter-Walkthrough) tutorial and although I managed to get the metric example working, it was rough given that setup information is scattered across many pages with sometimes confusing or conflicting information. For example:

* The [Istio Dev Guide](https://github.com/istio/istio/wiki/Dev-Guide) suggests to start Minikube with:

```
minikube start --bootstrapper kubeadm
```

But [Minikube Platform SetUp](https://istio.io/docs/setup/kubernetes/platform-setup/minikube/) , linked from [Istio Quick Start](https://istio.io/docs/setup/kubernetes/quick-start/) suggests to start with:
```
minikube start --memory=8192 --cpus=4 --kubernetes-version=v1.9.4 \
    --extra-config=controller-manager.cluster-signing-cert-file="/var/lib/localkube/certs/ca.crt" \
    --extra-config=controller-manager.cluster-signing-key-file="/var/lib/localkube/certs/ca.key" \
    --extra-config=apiserver.admission-control="NamespaceLifecycle,LimitRanger,ServiceAccount,PersistentVolumeLabel,DefaultStorageClass,DefaultTolerationSeconds,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota" \
    --vm-driver=`your_vm_driver_choice`
```
* The [Istio Quick Start](https://istio.io/docs/setup/kubernetes/quick-start/) which is linked from the [Istio Dev Guide](https://github.com/istio/istio/wiki/Dev-Guide) says you need to load Custom Resource Definitions (below), but during my tests this was not needed.

```
kubectl apply -f install/kubernetes/helm/istio/templates/crds.yaml
```
Finally, I faced a couple of [bugs](https://github.com/istio/istio/issues/9459) that were handled very gracefully by the Istio folks.

Therefore I thought it was would be nice for me to give back to the community by writing this small walkthough.

## Setup

1. Follow the Istio Dev Guide up to and including [Setting Up Environment Variables](https://github.com/istio/istio/wiki/Dev-Guide#setting-up-environment-variables)

    Some Important observations:

    * Install Minikube but do not start it.

    * At some point you might be directed to the [Istio Quick Start Guide](https://istio.io/docs/setup/kubernetes/quick-start/)  but **nothing is there needed for this example**

2. Switch to the original MOPA Walkthgouh but in the section [Before you Start](https://github.com/istio/istio/wiki/Mixer-Out-of-Process-Adapter-Walkthrough#before-you-start), checkout the Istio branch with the necessary fixes. As of this writing the branch is [release-1.1](https://github.com/istio/istio/tree/release-1.1)


    **Notice that there is no need to compile Istio at all for this MOPA walkthrough.**

3. Start Minikube

```
minikube start --bootstrapper kubeadm
```
4. Follow the overall structure of the original MOPA walkthough but use the _mygrpcadapter_ from this repo.





