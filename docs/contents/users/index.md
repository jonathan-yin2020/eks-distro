## Starting a Cluster

Before you start your cluster, you need to get several container images.
Container images can be accessed either from the public ECR registry or
built from scratch.

### EKS Distro Container Images

You can pull the images that make up the EKS Distro from the Public ECR Gallery,
and a [pull-all.sh shell
script](https://github.com/aws/eks-distro/blob/main/development/pull-all.sh) is
provided to demonstrate how to do this. You can also browse the [EKS Distro
Container Repository](https://gallery.ecr.aws/?searchTerm=eks-distro&verified=verified)

### Building Your Own Container Images
See the [Build Guide](build.md) for more information about building your own
container images.

## Run the kops.sh script

Run the `kops.sh` script, and when prompted supply a FQDN name for your cluster
for a domain you control. Refer to the [kops
documentation](https://kops.sigs.k8s.io/getting_started/aws/) for
full instructions.

After you run the script, be sure to run the `export KOPS_STATE_STORE=...`
command specified, and edit the yaml file with the output the script provides.

You will then need to run the following commands:
```bash
# Set to your cluster name
CLUSTER_NAME=my-super-cool-cluster.mydomain.com
kops create -f ./$CLUSTER_NAME.yaml
kops create secret --name $CLUSTER_NAME sshpublickey admin -i ~/.ssh/id_rsa.pub
kops update cluster $CLUSTER_NAME --yes
kops validate cluster --wait 10m
cat << EOF > aws-iam-authenticator.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-iam-authenticator
  namespace: kube-system
  labels:
    k8s-app: aws-iam-authenticator
data:
  config.yaml: |
    clusterID: $CLUSTER_NAME
EOF
kubectl apply -f aws-iam-authenticator.yaml
```
You can verify the pods in your cluster are using the EKS Distro images by running
the following command:
```bash
kubectl get po --all-namespaces -o json | jq -r .items[].spec.containers[].image | sort -u
```
To tear down the cluster, run:
```bash
kops delete -f ./$CLUSTER_NAME.yaml --yes
```
