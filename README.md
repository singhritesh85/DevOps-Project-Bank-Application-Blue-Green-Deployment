# DevOps-Project-Bank-Application-Blue-Green-Deployment-Aws

Installation of SonarQube-Scanner, Nexus Artifacts Uploader and pipeline utility step plugin as shown in the screenshot attached below.
![image](https://github.com/user-attachments/assets/9c8168bc-8fd1-4dfb-ad22-9a0d98d5850d)

Steps to be followed for Installation of Ingress nginx controller, ArgoCD and ArgoCD CLI is as given below.
```
kubectl create ns ingress-nginx
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx -n ingress-nginx

After creating it you need to edit the service and provide ssl certificate details and etc. in annotations as written below:- 
=================================================================
service.beta.kubernetes.io/aws-load-balancer-backend-protocol: http
service.beta.kubernetes.io/aws-load-balancer-connection-idle-timeout: "60"
service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"
service.beta.kubernetes.io/aws-load-balancer-ssl-cert: arn:aws:acm:us-east-2:XXXXXXXX:certificate/XXXXXX-XXXXXXX-XXXXXXX-XXXXXXXX
service.beta.kubernetes.io/aws-load-balancer-ssl-ports: https
service.beta.kubernetes.io/aws-load-balancer-type: elb

===================================================================
You need to change the targetPort for https to 80 in nginx ingress controller service as written below:-
-------------------------------------------------------------------------------------------------------------------------------
Before:

  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: http
    - name: https
      port: 443
      protocol: TCP
      targetPort: https
After:

  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: http
    - name: https
      port: 443
      protocol: TCP
      targetPort: 80
=================================================================

kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml


You can get the password for ArgoCD Login using the command as given below

kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d


Write the argocd-ingress-rule.yaml provide the service name and service port accordingly file as written below:-

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  namespace: argocd
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"    ###  You can use this option for this particular case for ArgoCD but not for all
#    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
#  ingressClassName: nginx
  rules:
  - host: argocd.singhritesh85.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: argocd-server   ### Provide your service Name
            port:
              number: 80     #### Provide your service port for this particular example you can also choose 443

----------------------------------------------
Installation of ArgoCD CLI
----------------------------------------------
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64

```

After Running the Jenkins Job the screenshot for SonarQube, Nexus and ArgoCD is as shown in the screenshot attached below.
![image](https://github.com/user-attachments/assets/9ccf6f5d-536f-4d87-b215-1e91e014d1af)
![image](https://github.com/user-attachments/assets/e9b919d3-21dd-406e-8db0-5c760b5dd32b)
![image](https://github.com/user-attachments/assets/8a5e6c10-b1bf-472b-ade2-f087e3f1bc84)


Install Argo Rollout Controller
```
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml
```
<br><br/>
Install Argo Rollouts Kubectl plugin
```
curl -LO https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-linux-amd64
chmod +x ./kubectl-argo-rollouts-linux-amd64
sudo mv ./kubectl-argo-rollouts-linux-amd64 /usr/local/bin/kubectl-argo-rollouts
```
<br><br/>
Finally Promote the rollout
```
kubectl get ro -n <namespace>
kubectl argo rollouts get rollout <rollout-name> -n <namespace>
kubectl argo rollouts promote <rollout-name> -n <namespace>
```
![image](https://github.com/user-attachments/assets/fe8ed082-e195-4b3f-b627-414c273b7eaf)
![image](https://github.com/user-attachments/assets/e74959c5-f8ea-4033-a5cd-f021e763bcae)
![image](https://github.com/user-attachments/assets/5f910d6a-7d95-4cfb-a2c9-0e526ca5abbf)
