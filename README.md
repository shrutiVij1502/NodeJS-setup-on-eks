### Setup AWS-managed Kubernetes Server
### Create Node JS Deployment and Service
### PV & Storage class
### Create Ingress Controller and Access Above created deployment using Ingress and point it to the domain (application should be accessible through a browser) 
### HPA & Probs
### SSL certificate
### kustomization

``` Step 1 - Setup AWS-managed Kubernetes Server ```

https://www.youtube.com/watch?v=1O-I7NtwCeE

Launch AWS CloudShell 

![image](https://user-images.githubusercontent.com/67600604/183015507-63ab72d2-7233-4357-87d8-d5528d365b35.png)

Download eksctl - 

``` curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp ```

Move eksctl to /usr/local/bin or /home/cloudshell-user/bin folder

``` sudo mv /tmp/eksctl /usr/local/bin ```

Test the installation by checking the version - ```eksctl version```

![image](https://user-images.githubusercontent.com/67600604/183016814-58c676fb-274d-4dfb-9a65-02d2ebc8ec24.png)

we can create the cluster manually or with the CLI, here, I created the cluster manually, for the CLI setup ypu can refer this Document 

https://aws.plainenglish.io/setting-up-amazon-eks-cluster-in-the-fastest-and-easiest-way-b5de835c28c3

Download kubectl

```curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.21.2/2021-07-05/bin/linux/amd64/kubectl ```

Apply execute permission

```chmod +x ./kubectl```

Move the kubectl to different folder and add it to the path

```mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin```

(Optional) Add the $HOME/bin path to your shell initialization file so that it is configured when you open a shell.

``` echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc```

Verify kubectl

``` kubectl version --short --client ```

![image](https://user-images.githubusercontent.com/67600604/183017660-84f87b44-e1ab-42ff-9289-f113ba4f713d.png)

``` kubectl get nodes -o wide ```

``` kubectl get pods --all-namespaces -o wide ``` 

![image](https://user-images.githubusercontent.com/67600604/183017829-5bf17771-7ac1-44dd-b358-d859cdf14533.png)

we can see that two nodes (ec2) are created with the EKS

![image](https://user-images.githubusercontent.com/67600604/183345106-fae0c127-effc-437c-a975-b9af00f352b1.png)

``` Step 2 - Create Node JS Deployment and Service ```

create three files named as frontend.yml, fronted-svc.yml and backend.yml to setup our nodejs application and put the data in the files as files are attached in the repository

frontend.yml has the nodejs application that is containing all the frontend part and frontend-svc is creating a service for that application , backend.yml is creating the database for having the images and storage for the application

now, check the deployments using  the kubectl get deploy

![image](https://user-images.githubusercontent.com/67600604/183345827-55defc63-cc53-4488-92a5-f277bb058475.png)

``` Step 3 - Create Ingress Controller and Access Above created deployment using Ingress and point it to the domain (application should be accessible through a browser) ```

creating the Ingress file - create a ingress file , but before that , firstky we have to take a domain, I have taken the domain from the freenom.com and configured that domain from the Route53 using the nameservers and mentioned that domain in the ingress file using 

create the file using - ``` kubectl create -f ingress.yml ```

and check using the ``` kubectl get ingress ```

![image](https://user-images.githubusercontent.com/67600604/183346750-e585a57f-f9fa-4836-84e7-013659cce240.png)

you can check the ingress loadbalancer using ``` kubectl get svc```

and assign that LB address in Route53 DNS to route from the website to the domain and 

![image](https://user-images.githubusercontent.com/67600604/183349198-3262c1d5-c445-44ff-87c0-d399e873b057.png)

now create hit that domain that we have mentioned in the ingress file - in my case , it is page9.ml

``` Step 4 - HPA & Probes ```

for HPA , we have two methods to create the HPA, one is the deploying the yaml file  and the other one is to just run a command for the HPA 

``` oc autoscale dc/database --min 1 --max 5 --cpu-percent = 75 ```
 
both works the same 

add the block in the deployment file in the container column 

![image](https://user-images.githubusercontent.com/67600604/183376266-1aacec1c-aa61-40a0-b611-4d2aec91828a.png)

if you want to create with the yml file , refer this article - https://www.tutorialspoint.com/openshift/openshift_application_scaling.htm

now , you can check using the ```kubectl get hpa``` command

for more for HPA - https://drive.google.com/file/d/10Kf0oimf9YjtwPTLAFcsWZbh3aYqm3rq/view?usp=sharing

![image](https://user-images.githubusercontent.com/67600604/183364607-305648ac-0873-487a-ac87-5cb60943afbc.png)

for probes , add a column of Liveness in the deployemnet part - https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

![image](https://user-images.githubusercontent.com/67600604/183372878-bbe499f6-39c8-464f-8571-481be547e938.png)

```Step 5 SSL certificate ```

for this , we need to install some dependencies using the helm 

```
kubectl create namespace cert-manager
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager --namespace cert-manager --version v1.2.0 --set installCRDs=true
nano production_issuer.yaml

add the lines 

apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    # Email address used for ACME registration
    email: ""ur_email_address""
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      # Name of a secret used to store the ACME account private key
      name: letsencrypt-prod-private-key
    # Add a single challenge solver, HTTP01 using nginx
    solvers:
    - http01:
        ingress:
          class: nginx
```
add the host in the file with the host and cluster issuer

![image](https://user-images.githubusercontent.com/67600604/183375453-34384ab3-b028-45e8-b721-9965cf204435.png)
![image](https://user-images.githubusercontent.com/67600604/183377871-47323c74-9842-490f-89a9-659863b18ba4.png)

this is created with the file ci.yml


``` Step 6 - kustomization```

Kustomize relies on the following system of configuration management layering to achieve reusability:

Base Layer
Specifies the most common resources
Patch Layers
Specifies use case specific resources

we can create a template as per  our requirements and can change as per we need the changes

like I have the deployment , svc and HPA file in the base folder and I want that same for the another folder but want HPA to be diffrent .so , I will take the other file as template from the base folder and patch the new HPA file with it 

In overlays folder , I am having my template Kustomization file 

![image](https://user-images.githubusercontent.com/67600604/183380165-2e1b83b9-d762-4277-978b-347c9c5de193.png)

and in base , I have the original ones 

![image](https://user-images.githubusercontent.com/67600604/183381868-7d319dfc-6c8c-4615-a46f-bc3539f9c052.png)


