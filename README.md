### Setup AWS-managed Kubernetes Server
### Create Node JS Deployment and Service
### PV & Storage class
### Create Ingress Controller and Access Above created deployment using Ingress and point it to the domain (application should be accessible through a browser) 
### HPA & Probs
### SSL certificate
### kustomization

``` Step 1 - Setup AWS-managed Kubernetes Server ```

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

creating the Ingress file - create a ingress file 
