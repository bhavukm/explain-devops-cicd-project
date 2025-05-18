# explain-devops-cicd-project
**How to explain a DevOps CICD project in an interview**

**Interview Question:** Explain the workflow of an End-To-End DevOps CICD Project that you have managed.

**Answer:-**

**Explain your company’s business.** For example, Critical Alert Systems, Application for Lawyers, an e-commerce website, a travel portal, etc.

**Application programming language:** Java Springboot framework, Python, Drupal, etc.

In this example, I am using the following tool stack/methodology:

We are using a GitLab source code repository, which will have all application source code files, Dockerfiles to build a container image, and then update the helm chart and push both to the AWS ECR repository.

Once the AWS ECR repository has the new helm chart, ArgoCD will deploy that to the AWS EKS Cluster.

For deploying our infrastructure, we utilize Terraform IaC that has been integrated within our CI/CD pipeline to create or modify infrastructure depending on the pipeline stage.

**DevOps Stages explanation:**

![image](https://github.com/user-attachments/assets/beba5356-36cf-4fa6-8944-f6a72f7d269d)


**Stage 1: Platform checks (Kubernetes Cluster)**

If I am deploying it to Dev, then my AWS EKS Cluster in the dev account should be up and running, the worker nodes should be in Ready state, the critical system components (like CoreDNS, kube-proxy, api-server, etc) should be working.

**Stage 2: Validate (Dockerfile, Kubernetes syntax checks, policies, Snyk SAST [Static Application Security Testing])**

Now, this stage will test whether your Dockerfile is present in the repo and any Kubernetes-based YAML manifest, including a helm chart (if using one), are correct syntax-wise. 

It can also test the Kubernetes policies. We may tell that we are using a tool like Kyverno, which can restrict stuff like “Restrict creation of pods with elevated permissions or Does the pod satisfy resource limits?”

Next, we have integrated the Snyk tool within our pipeline to perform SAST on our code to identify code vulnerabilities

**Stage 3: build artifact (maven or gradle)**

A build artifact is the final packaged version of your application, ready to be deployed. It could be a .jar file, .war file, .zip, .tar.gz, or even a Docker image.

In the third stage of our CI/CD pipeline, once the code has passed code quality checks and unit tests, we initiate the build process. We use Maven (or Gradle, depending on the project) to compile the source code, resolve dependencies, and generate a build artifact, typically a .jar or .war file. This artifact becomes the core output of the build phase and is what we deploy to our dev, test, and production environments.

Maven and Gradle are tools that automate the process of building Java applications. It:

•	Compile your code

•	Resolve and download any external libraries your app needs (called dependencies)

•	Run unit tests

•	Package everything into a deployable file (artifact)

**Stage 4: package (build Dockerfile, build Helm chart, add meta to image, push to AWS ECR)**

•	build_dockerfile - Containerizing the App

•	add meta to image - tag your docker image: use variables like $AWS_ECR_URI:$commit_id

•	Build helm chart – Preparing Kubernetes Deployment

•	push image and chart to AWS ECR

**Stage 5: Scan container image with Snyk**

•	Check if any of the software packages inside the image have known security issues (CVE IDs).

•	See if the image has any outdated libraries or risky dependencies.

•	Report anything suspicious, just like how antivirus software shows a list of threats.

**Stage 6: Promote to Dev cluster**

•	Create a config.yaml file with terraform version and helm-chart version:
_service:
  service-name:
     version: helm-chart-version
terraform:
     version: tf-version_

•	Push the config to another GitLab repo.

•	ArgoCD detects a new Helm chart version and starts to deploy it to the dev cluster

•	Run post-deployment tests if deployed successfully

•	Check on ArgoCD UI the status of pods, services, statefulsets, configmaps, etc.

If we need to deploy the dev helm chart to qa (test) environment, we will create a new configuration file for qa and update the version there. This will trigger another pipeline for qa, which takes the dev helm version and deploys to the AWS EKS cluster in qa AWS account.

The folder structure of the repo will be something like this:

![image](https://github.com/user-attachments/assets/ea816416-3eb2-4d56-ae3f-3596f10758e5)


Similarly, to deploy the qa version to prod, we will have another file for prod configuration, and it will trigger a production pipeline. However, it will also have a **Manual Approval** stage to approve the deployment. So, we can say that we are using **Continuous Delivery** methodology and **NOT Continuous Deployment** in production.

**What do you still need to learn so that you can answer questions on this project?**

1. How do you use GitLab templates to deploy infrastructure with Terraform and AWS EKS?

2. How do we authenticate AWS ECR repositories with ArgoCD?

3. How do we replicate AWS ECR Repositories across accounts?

4. How to set up a single GitLab server to run pipelines across different AWS accounts?

5. How do you manage secrets in CI/CD pipelines?

6. How do you remediate security vulnerabilities found while running the pipeline?

7. How do you manage the Terraform state file?
