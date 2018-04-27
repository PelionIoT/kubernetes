Adapting kubelet to be integrated into the wigwag system:

Components in kubelet which listen to apiserver for pod spec changes:
    Line 689: k8s.io/kubernetes/cmd/kubelet/app/server.go
        I'm pretty sure these lines will call CreateAndInitKubelet which then calls kubelet.NewMainKubelet which then calls makePodSourceConfig which then calls NewSourceApiserver
    	if builder == nil {
            builder = CreateAndInitKubelet
        }
        if kubeDeps.OSInterface == nil {
            kubeDeps.OSInterface = kubecontainer.RealOS{}
        }

        k, err := builder(kubeCfg, kubeDeps, &kubeFlags.ContainerRuntimeOptions, kubeFlags.HostnameOverride, kubeFlags.NodeIP, kubeFlags.ProviderID, kubeFlags.CloudProvider, kubeFlags.CertDirectory, kubeFlags.RootDirectory)
        if err != nil {
            return fmt.Errorf("failed to create kubelet: %v", err)
        }
    Line 285: k8s.io/kubernetes/pkg/kubelet/kubelet.go
        config.NewSourceApiserver(kubeDeps.KubeClient, nodeName, cfg.Channel(kubetypes.ApiserverSource))
        ** Also in this file statusManager syncs pod status back to api server
        ** livenessManager monitors pod health
        ** probeManager:
            // Manager manages pod probing. It creates a probe "worker" for every container that specifies a
            // probe (AddPod). The worker periodically probes its assigned container and caches the results. The
            // manager use the cached probe results to set the appropriate Ready state in the PodStatus when
            // requested (UpdatePodStatus). Updating probe parameters is not currently supported.
            // TODO: Move liveness probing out of the runtime, to here.
    Line 34: k8s.io/kubernetes/pkg/kubelet/config/apiserver.go
        lw := cache.NewListWatchFromClient(c.Core().RESTClient(), "pods", metav1.NamespaceAll, fields.OneTermEqualSelector(api.PodHostField, string(nodeName)))
    Line 34: k8s.io/kubernetes/vendor/k8s.io/client-go/tools/cache/listwatch.go

    Strategy for integration: Replace implementation of ListWatcher with one that talks to a wigwag server component like kube-bridge

