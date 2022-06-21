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






## Metrics Server

helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/

helm show values metrics-server/metrics-server
