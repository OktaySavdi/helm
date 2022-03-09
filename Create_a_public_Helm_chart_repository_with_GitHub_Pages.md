
# Create a public Helm chart repository with GitHub Pages

# **Step-by-step guide**

_Please keep in mind that following the instruction below, your Helm charts repository will be public._

Let me guide you through how to implement this solution.

**Create a new repository on your Github account (i.e. “helm-chart”):**

Follow the link  [https://github.com/new](https://github.com/new)  and login, then create a new git repository as per image:

![image](https://user-images.githubusercontent.com/3519706/157405969-0d720e04-b87b-4321-a436-59d4029c9134.png)

Since it’s an empty repository, we can’t publish a GitHub Pages site for now, keep the browser tab open for later.

**Clone the repository**
```bash
git clone https://github.com/oktaysavdi/helm-chart.git && cd helm-chart/  
Cloning into ‘helm-chart’…  
remote: Enumerating objects: 3, done.  
remote: Counting objects: 100% (3/3), done.  
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0  
Unpacking objects: 100% (3/3), done.
```
**Create a helm chart from scratch (or copy your own)**

I’m going to use the directory  `stable`  for the sources of our charts. I’m creating a default chart just for testing purposes, you might want to copy into  `deployment`  your charts instead:
```bash
mkdir stable charts && cd stable/ && helm create stable/deployment
Creating stable/deployment
```
**Lint the chart**

As a good habit,  `helm lint`  runs a series of tests to verify that  the chart is well-formed:
```bash
helm lint stable/deployment/
```
**Create the Helm chart package**
```bash
helm package stable/deployment/ 
Successfully packaged chart and saved it to: /root/helm/deployment-0.1.0.tgz
```
Move the generated .tgz file to  `/charts`  by doing a  `mv deployment-0.1.0.tgz charts/`
```bash
mv /root/helm/helm/deployment-0.1.0.tgz charts/
```
**Create the Helm chart repository index**

According to  [Helm](https://helm.sh/docs/developing_charts/):

> A repository is characterized primarily by the presence of a special file called `index.yaml` that has a list of all of the packages supplied by the repository, together with metadata that allows retrieving and verifying those packages.

So, everything we need to create is the  `index.yaml`  file and the command to do that is:
```bash
helm repo index charts/ --url https://oktaysavdi.github.io/helm/charts/
```
```bash
cat index.yaml   
apiVersion: v1  
entries:  
  deployment:  
  - apiVersion: v2  
    appVersion: 1.16.0  
    created: "2022-03-08T19:02:33.037670059+03:00" 
    description: A Helm chart for Kubernetes  
    digest: 7f7e31710c3b525dbcb3649ef26e955df85c76141f990abe77b2791965ae84eb
    name: deployment
    type: application
    urls:  
    - https://oktaysavdi.github.io/helm/charts/deployment-0.1.0.tgz 
    version: 0.1.0  
generated: "2022-03-08T19:02:33.03620579+03:00"
```
**Add exclude list**
```bash
vi _config.yaml
exclude:
  - "README.md"
```
**Push the git repository on GitHub**
```bash
git add . && git commit -m “Initial commit” && git push origin master
```
**Configure the “helm-chart” repository as a Github pages site**

Now it’s time to publish the contents of our git repository as Github pages. Go back to your browser, in the “settings” section of your git repository, scroll down to Github Pages section and configure it as follow:

![image](https://user-images.githubusercontent.com/3519706/157408400-b21ffbbe-0f3c-4943-9584-342395024164.png)

Check github page work properly

![image](https://user-images.githubusercontent.com/3519706/157408685-6d5d7656-665d-45c3-8d60-74e716e4bf3b.png)

**Test the github pages**

![image](https://user-images.githubusercontent.com/3519706/157411853-8800536e-7dbb-4bb6-a7ce-689002ae2dda.png)


**Configure helm client**

In order to share your brand new repository, everyone interested in your charts need to configure their own Helm client. Also on client side, repositories are managed with the  `$ helm repo`  commands:
```bash
helm repo add app-stable https://oktaysavdi.github.io/helm/charts/
“app-stable” has been added to your repositories
```
**Test the Helm chart repository**
```bash
helm search app-stable  
NAME CHART VERSION APP VERSION DESCRIPTION  
[…]  
stable/deployment 0.1.0 A Helm chart for Kubernetes  
[…]
```

**Reference**:

+ https://medium.com/@mattiaperi/create-a-public-helm-chart-repository-with-github-pages-49b180dbb417
+ https://tech.paulcz.net/blog/creating-a-helm-chart-monorepo-part-1/
