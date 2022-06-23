kubectl patch deployment coredns \
    -n kube-system \
    --type json \
    -p='[{"op": "remove", "path": "/spec/template/metadata/annotations/eks.amazonaws.com~1compute-type"}]'








kubectl \
   --server="https://FD5D71BC420BDF2D4CAFD7E280D0D0B6.gr7.us-east-1.eks.amazonaws.com" \
   --certificate_authority=/tmp/ca.crt \
   --token="k8s-aws-v1.aHR0cHM6Ly9zdHMuYW1hem9uYXdzLmNvbS8_QWN0aW9uPUdldENhbGxlcklkZW50aXR5JlZlcnNpb249MjAxMS0wNi0xNSZYLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFXRlVSRUpBNVZXVUQ2QVlGJTJGMjAyMjA2MjElMkZ1cy1lYXN0LTElMkZzdHMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDIyMDYyMVQwODA3MjFaJlgtQW16LUV4cGlyZXM9MCZYLUFtei1TaWduZWRIZWFkZXJzPWhvc3QlM0J4LWs4cy1hd3MtaWQmWC1BbXotU2lnbmF0dXJlPWViYjNlOTkwZTRmNGNmZjIzMjE5N2UyYzY2ZGMzNzk0OWNkOTJkNjc1MDg3YWJiZmY1NGQyNTFiYTUxYzhhYzk" \
   patch deployment coredns \
   -n kube-system --type json \
   -p='[{"op": "remove", "path": "/spec/template/metadata/annotations/eks.amazonaws.com~1compute-type"}]'









## 1 Simple Deployment
- create `8-fargate-staging.tf`
- create `k8s/simple-deployment.yaml`
kubectl get events -w -n staging
kubectl apply -f k8s/simple-deployment.yaml
kubectl get pods -n staging
kubectl get nodes


## 2 Metrics Server

helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/

helm show values metrics-server/metrics-server

kubectl top pods -n staging

## 4 Automaticlly scale based on cpu or memory

- create `k8s/service.yaml`
kubectl apply -f k8s/service.yaml

- create `k8s/hpa.yaml`
kubectl apply -f k8s/hpa.yaml
kubectl get hpa -n staging


watch -n 1 kubectl get pods -n staging
kubectl get hpa php-apache -w -n staging
kubectl run -i --tty -n staging load-generator --pod-running-timeout=5m0s --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"

## PDB

- create `k8s/pdb.yaml`
kubectl apply -f k8s/pdb.yaml
kubectl get pdb -n staging

## OpenId conect

- create `terraform/10-iam-oidc.tf`
terraform init
terraform apply

## Deploy AWS Load balancer controller

create `11-iam-controller.tf`
create `AWSLoadBalancerController.json`
create `12-helm-controller.tf`

terraform apply
create `k8s/ingress.yaml`

kubectl logs -f -n kube-system \
-l app.kubernetes.io/name=aws-load-balancer-controller

kubectl apply -f k8s/ingress.yaml
kubectl get ing -n staging
create CNAME
dig +short php-apache.devopsbyexample.io

## Secure with TLS

issue a certificate
add
```
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-east-1:424432388155:certificate/9c3c828c-5d42-4b0e-b7f9-64fcfecb8af9
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS":443}]'
    alb.ingress.kubernetes.io/ssl-redirect: '443'
```
kubectl apply -f k8s/ingress.yaml
check lb from the console
check https://php-apache.devopsbyexample.io

## Create network load balancer

create k8s/lb.yaml
kubectl apply -f k8s/lb.yaml
kubectl get svc -n staging
check in the console
curl http://k8s-staging-phpapach-720344fc29-4d24ad27abc92c58.elb.us-east-1.amazonaws.com


## Storage

Currently, Fargate does not support PersistentVolume back by EBS. You can use EFS instead.
However, since the storage capacity is a required field by Kubernetes, you must specify the value and you can use any valid value for the capacity.

When using standard EC2 worker nodes, the EFS CSI driver needs to be deployed as a set of pods and DaemonSets. With this new update, for Fargate this step is not required and you do not need to install the EFS CSI driver, as it is installed in the Fargate stack and support for EFS is provided out of the box. Customers can use EFS with Fargate for EKS without spending the time and resources to install and update the CSI driver.

The current implementation of the EFS CSI driver requires the volumes to be statically pre-created for the PVC binding to work.
Check your VPC configuration. If you are using a custom VPC, make sure that DNS settings are enabled.

kubectl exec -n staging app -- stat /data

