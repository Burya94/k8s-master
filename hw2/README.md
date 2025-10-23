## AWS Cloud Controller Manager

### Steps

* Clone [AWS CCM source code repo](https://github.com/kubernetes/cloud-provider-aws):
```
git clone https://github.com/kubernetes/cloud-provider-aws.git
```
* Build CCM localy:
```
make aws-cloud-controller-manager
```

* Download IMDS mock(https://github.com/aws/amazon-ec2-metadata-mock):
```
wget https://github.com/aws/amazon-ec2-metadata-mock/releases/download/v1.13.0/ec2-metadata-mock-linux-amd64 && \
chmod +x ec2-metadata-mock-linux-amd64
```

* Configure routing for metadata server:
```
sudo ip route del 169.254.169.254/32
sudo ip route add 169.254.169.254/32 via 127.0.0.1
sudo iptables -t nat -A OUTPUT -d 169.254.169.254 -j DNAT --to-destination 127.0.0.1
```
* Start Kubernetes cluster with:
```
./setup.sh start
```

* Start meta-data mock server
```
sudo ./ec2-metadata-mock-linux-amd64 --port=80
```

* Gerenate temporary AWS credentials for CCM and export to root env
```
export AWS_REGION=eu-central-1
export AWS_ACCOUNT_ID=
export AWS_ACCESS_KEY_ID=
export AWS_SECRET_ACCESS_KEY=
export AWS_SESSION_TOKEN=
```
* Create a cloud config for CCM
```
cat << EOF >> cloud.config
[Global]
Region                                          = us-east-1
VPC                                             = <VPC ID where the cluster is installed>
SubnetID                                        = <Single subnet ID used by load balancer controller>
KubernetesClusterTag                            = <kubernetes cluster ID>
DisableSecurityGroupIngress                     = false
ClusterServiceLoadBalancerHealthProbeMode       = Shared
ClusterServiceSharedLoadBalancerHealthProbePort = 0
EOF
```
* CCM
```
sudo ./aws-cloud-controller-manager -v=2     --cloud-config="./cloud.config"     --kubeconfig="/root/.kube/config"     --cloud-provider=aws  --configure-cloud-routes=false     --leader-elect=false
```

* Create deployment and expose it via LoadBalancer

```
sudo ./kubebuider/bin/kubectl create -f nginx_deployment.yaml
sudo ./kubebuilder/bin/kubectl expose deployment nginx --port=80 --target-port=80 --type=LoadBalancer
```

* Verify LoadBalancer is created and obtain its IP

```
sudo apt update -y && sudo apt install dnsutils -y

sudo ./kubebuilder/bin/kubectl get service nginx -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
dig  +short
```