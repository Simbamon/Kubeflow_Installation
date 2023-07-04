# Kubeflow_Installation

## Installing the Kubeflow in Kubernetes-Homelab
- Kubernetes Spec
    - Linux: Ubuntu
    - CNI: Weave
    - 2 Masters, 1 LB, 2 Workers
1. Install LVM
    ```bash
    sudo apt-get install -y lvm2
    ```
2. Clone Rook Github Repo
    ```bash
    mkdir rook
    cd rook
    git clone --single-branch --branch release-1.5 https://github.com/rook/rook.git
    ```
3. Install Rook
    - Install the CRDs
    ```bash
    cd rook/cluster/examples/kubernetes/ceph
    kubectl create -f crds.yaml -f common.yaml -f operator.yaml
    ```
    - Create the cluster
    ```bash
    kubectl create -f cluster-test.yaml
    ```
    - Validating
    ```bash
    kubectl -n rook-ceph get pod
    watch kubectl get pods -n rook-ceph
    ```
4. Storage Class Provisioning
    - Use storageclass-test.yaml from Rook Github Repo (rook/cluster/examples/kubernetes/ceph/csi/rdb)
    ```bash
    kubectl apply -f storageclass-test.yaml
    ```
5. Make the storage class as default
    ```bash
    kubectl patch sc rook-ceph-block -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
    ```
6. Install Kubeflow
    - Prerequisites
        - Kubernetes (up to 1.25) with a default StorageClass
        - kustomize 5.0.0  
            ⚠️ Kubeflow is not compatible with earlier versions of Kustomize. This is because we need the sortOptions field, which is only available in Kustomize 5 and onwards #2388.
        - kubectl
    - Download Kustomize
        ```bash
        sudo apt-get kustomize
        ```
    - Create a new directory and clone the Kubeflow manifests repo
        ```bash
        mkdir kf
        cd kf
        git clone https://github.com/kubeflow/manifests.git
        ```
    - Install with a single command
        ```bash
        while ! kustomize build example | awk '!/well-defined/' | kubectl apply -f -; do echo "Retrying to apply resources"; sleep 10; done
        ```
    - Check the installation in different terminal
        ```bash
        watch kubectl get pods -A
        ```
7. Check all Kubeflow-related Pods are ready
    ```bash
    kubectl get pods -n cert-manager
    kubectl get pods -n istio-system
    kubectl get pods -n auth
    kubectl get pods -n knative-eventing
    kubectl get pods -n knative-serving
    kubectl get pods -n kubeflow
    kubectl get pods -n kubeflow-user-example-com
    ```
8. Port-Forward
    - Run the following to port-forward Istio's Ingress-Gateway to local port 8080
    ```bash
    kubectl port-forward svc/istio-ingressgateway -n istio-system 8080:80
    ```