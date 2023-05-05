# *CAPSTONE : containerized microservices app*

### Note for the Reviewer: please check the Project Rubric below and `screenshots/` directory.

## Scope of the Project

### What is this?

It is a small dockerized software application to be used as a starting point
for the further projects based on Python FastAPI framework deployed on AWS EKS, based on a single
container or multi-container setup. It is based on a basic Poetry skeleton
and Uvicorn (an app server supporting the ASGI protocol).

### Who’s it intended for?

Any developer who wants to quickly start building an application on the framework
described above, ultimately making it easily available with the help of AWS EKS.

### What problem does the software solve?

Provides a simple, scalable starting point for a further development of applications
using Poetry and Uvicorn, which is often quite problematic in the context of building
containers on this set. Additionally, the given project shows step by step how to make
the API public and how to configure CICD based on GitHub Actions and Docker Hub.

### How is it going to work? What to do to run it?

0. Use CTRL+F to override user specific paths (find all `ozieblo` related) or names like `capstone` if needed.
1. The app skeleton has two routes available: `/` and `/hello/{name}` , unit tested by `test_main.http`.
It is a default for FastApi starting the project in PyCharm.

2. Configure AWS credentials for the dedicated user. Check `screenshots/IAM.png`. Configure RSA key.

3. Sign in to Docker.

4. Follow manuals if you want to run the app on localhost:
    - https://python-poetry.org/docs/cli/
    - https://fastapi.tiangolo.com/deployment/manually/

5. Use `docker-compose up` if you want to run the container on localhost (Uvicorn running on http://0.0.0.0:8000).

6. In terminal, run `kubectl version --short` and `aws --version` to confirm that all works fine.

7. Update `env-configmap.yml`, and save all your configuration values (non-confidential environments variables) in that file.

8. Create `aws-secret.yml` file to store your AWS login credentials. Replace `credentials`
   with the Base64 encoded credentials (not the regular username/password). You may use the following command:
   `cat ~/.aws/credentials | tail -n 2 | base64`

9. Check or update `eks-cluster/deployment.yml` file individually for each service. While defining container specs, make sure to specify
the same images you've pushed to the DockerHub. Ultimately, the application
should run in respective pods. Similarly, create the `eks-cluster/service.yml` file thereby defining the right services/ports mapping.

10. EKSCTL is a command-line utility to create an EKS cluster and the associated resources in a single command.
Create cluster using `eksctl create cluster --name capstone --region=us-east-1 --nodes-min=2 --nodes-max=3`
The command will set the following for you:
    - Two m5.large worker nodes. Recall that the worker nodes are the virtual machines, and the m5.large type defines that
    each VM will have 2 vCPUs, 8 GiB memory, and up to 10 Gbps network bandwidth.
    - Use the Linux AMIs as the underlying machine image
    - An autoscaling group with [2-3] nodes
    - Importantly, it will write cluster credentials to the default config file locally. Meaning, EKSCTL will set up KUBECTL
    to communicate with your cluster. If you'd have created the cluster using the web console, you'll have to set up the kubeconfig manually.

11. Apply env variables, secrets, deploymend and service configuration (do the same for other services if developed):
    - `kubectl apply -f env-configmap.yml`
    - `kubectl apply -f aws-secret.yml`
    - `kubectl apply -f eks-cluster/deployment.yml`
    - `kubectl apply -f eks-cluster/service.yml`

12. Check `kubectl get pods` , use `kubectl describe pod/POD_NAME` if there is any issue.

13. Autoscale, if required, as `kubectl autoscale deployment backend-user --cpu-percent=70 --min=3 --max=5`
or follow https://github.com/kubernetes-sigs/metrics-server and run commands presented on `screenshots/hpa.png`.

14. Create an external load balancer and assign a fixed, external IP to the Service:
`kubectl expose deployment capstone --type=LoadBalancer --name=publiccapstone  --port=8000`.

15. Run `kubectl get services` to obtain external IP. Check `screenshots/Postman.png`.


### What are the main concepts that are involved?

- https://fastapi.tiangolo.com
- https://python-poetry.org/docs/
- https://stackoverflow.com/questions/66362199/what-is-the-difference-between-uvicorn-and-gunicornuvicorn
- https://docker-curriculum.com/#getting-started
- https://docs.docker.com/compose/
- https://www.docker.com/products/docker-hub/
- https://docs.github.com/en/actions
- https://aws.amazon.com/eks/
- https://kubernetes.io/docs/reference/kubectl/
- https://eksctl.io/introduction/
- https://aws.amazon.com/elasticloadbalancing/
- https://aws.amazon.com/cloudwatch/

### What are the main potential issues during recycling of the given project?

- In case of problems when hosting in a region other than us-east-1,
check if it is not local only to it.
- Watch for defined port consistency!
- Consider to clean `.kube` directory if there is an issue with kubectl credentials.
- Do not forget about RSA key.

### What technical details need developers to know to develop the software or new feature?

The project does not show AWS S3 configuration or database connection, but their
implementation is possible on this base. In the future, the project may be expanded
with EKSCTL and Serverless/Terraform configuration files.

### Are there specific coverage goals for the tests?

Simple confirmation of the availability and functionality of the indicated endpoints
both at the localhost and public access level. Configuration correctness based on data
monitoring from the console, available to check on screenshots.

## Project Rubric

### CI/CD, Github & Code Quality

1. The project demonstrates an understanding of CI and Github.

All project code is stored in this GitHub repository and links to the repository
has been provided for the review. GitHub Actions used
to check and build the application (check .github/workflows/). Please refer to screenshots included in the repository.

2. The project has a proper documentation.

The README file includes introduction how to setup and deploy the project. It explains
the main building blocks and has important comments.

3. The project use continuous deployments (CD).

A CD tool is in place to deploy new version of the app automatically to production via Docker Hub
and defined EKS configuration. The way is described and easy to follow.

### Container

1. The app is containerized

There is a Dockerfile in repo and the docker image(s) can be build based on Docker Compose.

2. The project have public docker images.

On DockerHub image for the application is available: https://hub.docker.com/repository/docker/ozieblomichal/capstone/general

3. The application runs in a container without errors.

The app runs as a container on a local system without errors.

### Deployment

1. The application runs on a cluster in the cloud.

The project can be deployed to a kubernetes cluster

2. The app can be upgraded via rolling-update.

The students can deploy a new version of the application without downtime.

3. A/B deployment of the application.

Two versions of the same app can run at the same and service traffic

4. Monitoring.

The application is monitored by Amazon CloudWatch.
