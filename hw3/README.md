### Node registration with a CSR


## Start Control Plane components

```
./setup.sh start
```

## create a default clusterrolebinding for node bootstrap


```
sudo ./kubebuilder/bin/kubectl create clusterrolebinding kubelet-bootstrap \
  --clusterrole=system:node-bootstrapper \
  --group=system:bootstrappers
```

### create a bootstrap token secret

```
sudo ./kubebuilder/bin/kubectl apply -f secret.yaml
```

```
sudo cp /var/lib/kubelet/kubeconfig /var/lib/kubelet/bootstrap-kubeconfig
sudo ./kubebuilder/bin/kubectl config set-credentials tls-bootstrap-token-user --token=07401b.f395accd246ae52d --kubeconfig=/var/lib/kubelet/bootstrap-kubeconfig
sudo ./kubebuilder/bin/kubectl config set-context tls-bootstrap-token-user@test-env --cluster=test-env --user=tls-bootstrap-token-user --kubeconfig=/var/lib/kubelet/bootstrap-kubeconfig
sudo ./kubebuilder/bin/kubectl config use-context tls-bootstrap-token-user@test-env --kubeconfig=/var/lib/kubelet/bootstrap-kubeconfig
sudo mkdir ls /var/lib/kubelet/newnode


sudo PATH=$PATH:/opt/cni/bin:/usr/sbin kubebuilder/bin/kubelet \
            --bootstrap-kubeconfig=/var/lib/kubelet/bootstrap-kubeconfig \
            --kubeconfig=/var/lib/kubelet/kubeconfig \
            --config=/var/lib/kubelet/config.yaml \
            --root-dir=/var/lib/kubelet \
            --cert-dir=/var/lib/kubelet/pki \
            --hostname-override=$(hostname) \
            --pod-infra-container-image=registry.k8s.io/pause:3.10 \
            --node-ip=$HOST_IP \
            --cloud-provider=external \
            --cgroup-driver=cgroupfs \
            --max-pods=4  \
            --v=1 
```

```
sudo ./kubebuilder/bin/kubectl get csr
```
