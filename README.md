# LogEntry Mixer Out of Process Adapter (MOPA) Walkthrough

This is an adaptation and expansion of the [Mixer Out of Process Adapter (MOPA) Walkthrough](https://github.com/istio/istio/wiki/Mixer-Out-of-Process-Adapter-Walkthrough).

## Why this Guide

The MOPA walkthrough is very nice, but the prep work needed before actually going through the example is very rough. Information is scattered across many pages with sometimes confusing or conflicting information. For example:

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

Finally, I faced a couple of [bugs](https://github.com/istio/istio/issues/9459) that were handled very graciously by the Istio folks.

Therefore I thought it would be nice to give back to the community by writing this small guide.

## Target Audience

This guide is a summary and adaption of many other existing pages. It is for people that want to experiment and get their feet wet with MOPAs but not necessarily push code back to Istio.

Furthermore this guide was tested with Minikube running on MacOS with VirtualBox installed.

## Background

I suggest you read the page on [Policies and Telemetry](https://istio.io/docs/concepts/policies-and-telemetry/).

## Setup

1. We will start with the original [MOPA Walkthrough](https://github.com/istio/istio/wiki/Mixer-Out-of-Process-Adapter-Walkthrough) which will direct us to the Istio Dev Guide

2. Follow the Istio Dev Guide up to and  including [Other Dependencies](https://github.com/istio/istio/wiki/Dev-Guide#other-dependencies). Do not go further.

   * At some point you might be directed to the [Istio Quick Start Guide](https://istio.io/docs/setup/kubernetes/quick-start/)  but **nothing in there is needed for this example**

2. Install Minikube from the [official documentation](https://kubernetes.io/docs/tasks/tools/install-minikube/) .

    * Start Minikube:

        ```
        minikube start --bootstrapper kubeadm
        ```
3. Switch back to the Istio Dev Guide and follow _only_ the section [Setting up environment variables]https://github.com/istio/istio/wiki/Dev-Guide#setting-up-environment-variables

3. Now go back to the original MOPA Walkthgouh but in the section [Before you Start](https://github.com/istio/istio/wiki/Mixer-Out-of-Process-Adapter-Walkthrough#before-you-start), checkout the Istio branch with the necessary fixes. As of this writing the branch is [release-1.1](https://github.com/istio/istio/tree/release-1.1). In other words, it should read:

    ```
    mkdir -p $GOPATH/src/istio.io/ && \
    cd $GOPATH/src/istio.io/  && \
    git clone https://github.com/istio/istio.git --branch release-1.1
    ```

    **Notice that there is no need to compile Istio at all for this MOPA walkthrough.**

4. Follow the overall structure of the original MOPA walkthough but use the _mygrpcadapter_ from this repo.

# Testing

When Testing I use the following _mixc_ CLI in order to fully exercise the code. Notice that I am using MACOS.
```
$GOPATH/out/darwin_amd64/release/mixc report \
--timestamp_attributes request.time="2017-07-04T00:01:10Z" \
-s destination.service="svc.cluster.local",source.user=”kubernetes://nets-57cdb6d9d7-rj7jk.default”,\
request.method="POST",request.path=”/istio.mixer.v1.Mixer/Check”,\
request.scheme="https" -i request.size=1235,response.size=1024,response.duration=100 \
--bytes_attributes source.ip=ac:11:0:0d,destination.ip=ac:11:0:03
```

If want to see mixs debug logs, start it with:

```
$GOPATH/out/darwin_amd64/release/mixs server --configStoreURL=fs://$(pwd)/mixer/adapter/mygrpcadapter/testdata --log_output_level debug
```

If every goes okay you will see similar output

```
listening on "[::]:9000"
2018-10-30T08:05:55.488405Z info  received request {[&InstanceMsg{Variables:map[
string]*istio_policy_v1beta11.Value{destinationIp:
&Value{Value:&Value_IpAddressValue{IpAddressValue:&IPAddress{Value:[172 17 0 3],},},},latency:
&Value{Value:&Value_DurationValue{DurationValue:&Duration{Value:100ns,},},},method:
&Value{Value:&Value_StringValue{StringValue:POST,},},protocol:
&Value{Value:&Value_StringValue{StringValue:https,},},requestSize:
&Value{Value:&Value_Int64Value{Int64Value:1235,},},responseCode:
&Value{Value:&Value_Int64Value{Int64Value:0,},},responseSize:
&Value{Value:&Value_Int64Value{Int64Value:1024,},},sourceIp:
&Value{Value:&Value_IpAddressValue{IpAddressValue:&IPAddress{Value:[172 17 0 13],},},},sourceUser:
&Value{Value:&Value_StringValue{StringValue:”kubernetes://nets-57cdb6d9d7-rj7jk.default”,},},url:
&Value{Value:&Value_StringValue{StringValue:”/istio.mixer.v1.Mixer/Check”,},},},Timestamp:&istio_polic
y_v1beta11.TimeStamp{Value:2017-07-04T00:01:10Z,},Severity:Default,MonitoredResourceType:,
MonitoredResourceDimensions:map[string]*istio_policy_v1beta11.Value{},Name:i1logentry.instance.istio-system,}]
&Any{TypeUrl:type.googleapis.com/adapter.mygrpcadapter.config.Params,Value:[10 7 111 117 116 46 116
120 116],XXX_unrecognized:[],} 17679084362437296840}

TimeStamp:  2017-07-04T00:01:10Z
Severity:  Default
protocol :  https
sourceUser :  ”kubernetes://nets-57cdb6d9d7-rj7jk.default”
destinationIp :  172.17.0.3
requestSize :  1235
sourceIp :  172.17.0.13
responseCode :  0
latency :  100ns
url :  ”/istio.mixer.v1.Mixer/Check”
method :  POST
responseSize :  1024
2018-10-30T08:05:55.488756Z	info	writing instances to file out.txt

```

# Troubleshooting

If you see:

```
E1029 23:21:38.114345   93959 start.go:302]
Error restarting cluster:  restarting kube-proxy: waiting for
kube-proxy to be up for configmap update: timed out waiting
for the condition
```
It possibly means you are (or were) connected to a VPN. In my case only a computer restart would solve this issue. Minikube stop, delete and rm -rf ~/.kube would not solve it. Other suggestions?

For reference, this is the [github](https://github.com/kubernetes/minikube/issues/3022) issue with a long list of complains and possible solutions.

# References

 * This [page](https://github.com/istio/istio/wiki/Mixer-Running-a-Local-Instance) has a good (and maybe the only?) rich mixc example.

 * It is always good to have the [mixc reference](https://istio.io/docs/reference/commands/mixc/) bookmarked.

 * [Istio Dev Guide](https://github.com/istio/istio/wiki/Dev-Guide)

 * [MOPA Walkthrough](https://github.com/istio/istio/wiki/Mixer-Out-of-Process-Adapter-Walkthrough)

 * For some in-depth information on Istio Adapters you should read the [Mixer Compiled In Adapter Dev Guide](https://github.com/istio/istio/wiki/Mixer-Compiled-In-Adapter-Dev-Guide)

 * Minikube [Releases](https://github.com/kubernetes/minikube/releases) page

 * [Prometheus Out of Mixer Example](https://github.com/istio/istio/tree/master/mixer/test/prometheus). Useful after you've mastered the MOPA walkthough and are familiar with deploying apps in Kubernetes.

 * Background info on [Policies and Telemetry](https://istio.io/docs/concepts/policies-and-telemetry/) .

 * [Mixer Out Of Process Adapter Dev Guide](https://github.com/istio/istio/wiki/Mixer-Out-Of-Process-Adapter-Dev-Guide)

















