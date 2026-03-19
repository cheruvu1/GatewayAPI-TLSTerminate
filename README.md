Step1: Installing the Gateway API CRDs
kubectl kustomize "https://github.com/nginx/nginx-gateway-fabric/config/crd/gateway-api/standard?ref=v2.2.1" | kubectl apply -f –
Verify CRDs
kubectl get crd | grep gateway.networking.k8s.io

Step2: Deploy NGINX Gateway Fabric
helm install nginx-gateway-fabric oci://ghcr.io/nginx/charts/nginx-gateway-fabric --create-namespace -n nginx-gateway --set nginx.service.type=LoadBalancer
After installation, verify that the controller is running:
kubectl get GatewayClass 
kubectl get pods -n nginx-gateway 
kubectl get all -n nginx-gateway 
kubectl get pods,svc -n nginx-gateway





Step3: Deploy Gateway
git clone https://github.com/cheruvu1/GatewayAPI-TLSTerminate/blob/main/main-gateway.yaml
kubectl apply -f main-gateway.yaml
kubectl get gateway


Step4:  Deploy  HTTPRoute
kubectl apply -f HTTPRoute.yaml
kubectl get httproute



Step5: Deploy application

kubectl apply -f Deployment-v1.yaml

kubectl get deployments
kubectl get services

-	Note: main-gateway-nginx has port 80.
-	Capture the LB





Step6: Application Testing

Terminal:

export IP_VIP=$(kubectl get gateway main-gateway -o jsonpath='{.status.addresses[0].value}') 
echo $IP_VIP
curl -vvv --header "Host: backends.example" http://$IP_VIP:80

Browser:

http://backends.example:80



Step7: TLS Termination Configuration

openssl req -x509 -nodes -days 365 \ -newkey rsa:2048 \ -keyout tls.key \ -out tls.crt \ -subj "/CN=backends.example/O=demo"
kubectl create secret tls demo-cert --cert=tls.crt --key=tls.key
Open main-gateway.yaml file, uncomment the lines from 15 to 22


 
kubectl apply -f main-gateway.yaml
kubectl get gateway
kubectl get services

Note: main-gateway-nginx service will now have port 443

Step8: Secure traffic (HTTPS) Application Testing

Update windows host file location:
C:\Windows\System32\drivers\etc\hosts

export IP_VIP=$(kubectl get gateway main-gateway -o jsonpath='{.status.addresses[0].value}') 
echo $IP_VIP

<IP_VIP> backends.example

Example:
xx.xx.248.105 backends.example

export IP_VIP=$(kubectl get gateway main-gateway -o jsonpath='{.status.addresses[0].value}') 
echo $IP_VIP
curl -k https://backends.example:443 
curl -vvv --header "Host: backends.example" http://$IP_VIP:80

Browser:
https://backends.example:443

