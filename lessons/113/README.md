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