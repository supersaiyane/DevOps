:metal: This topic will be divided into a series where I, will be explaining different tools setup and integrations with one another to create a complete DevOps CI/CD pipeline.
We will be exploring the tools like

1. Gitlab
   - Merge Request and Code Review
   - Integration of Gitlab with Jenkins
2. Jenkins
   - Using Jenkins File
   - PollingSCM
   - Email Extensions(sending job finish emails)
   - Private Repo Pull/Push
   - Maintaining Docker Image Version
   - Blue Ocean plugin
3. Docker
4. Ansible
   - Automation of App Deployment
   - App Redeploying Strategy
5. Google Container Registry
   - Maintaining Container Registry
6. Google Hosted Kubernetes
   - Creating and using ConfigMaps with the deployed application.
   - Monitoring Kubernetes

**Additional**

1. Scaling Jenkins on Kubernetes.
2. Using Istio.
   - Ingress/Egress
   - Gateways
   - Virtual Services
   - Traffic Management
   - Canary Deployments
   - Service Mesh
     - Kiali
     - Prometheus
     - Jaeger
3. Using Nginx-Ingress controller for Ingress on kubernetes.
4. Spinnaker
5. DNS settings for Using Service name instead of Cluster IP.
6. Using Service Account Keys with GOOGLE_APPLICATION_CREDENTIALS
   - Production environment
   - Development environment

**Few Assumptions are always good :fox_face:**

1. _This complete tutorial will be created on Ubuntu 20.04 LTS(Focal Fossa)._
2. _You have Gitlab account._
   - _[You can create if not!](https://gitlab.com/-/trial_registrations/new?glm_source=about.gitlab.com&glm_content=free-trial)_
3. _You have signed up Google cloud services._
   - _You need to have Google account or [signup for a free account](https://console.cloud.google.com/freetrial/signup/)_
4. _You have Visual Code installed._
   - _[If not,get it from here](https://code.visualstudio.com/)_
5. _Know the basics of Git._
   - _[Basics](https://github.com/supersaiyane/Git-Cheetsheet)_
   - _[Using Git Plugin in Visual Code](https://code.visualstudio.com/docs/editor/versioncontrol)_
6. _Know the basics of Docker._
   - _[Basics](https://github.com/supersaiyane/docker-cheatsheet/blob/master/README.md)_
7. _We will be using React Application as Frontend and Backend as Nodejs server._
