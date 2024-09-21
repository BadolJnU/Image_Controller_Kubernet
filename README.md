# Image-Controller-Kubernets
A Kubernetes controller monitors applications (such as Deployments and DaemonSets) and "caches" public container images by uploading them to a private registry and updating the applications to use these new image copies.

## Project's Motivation
- We want to protect against the risk of public container images being removed from their registries, which could disrupt our deployments. 
- In my Kubernetes cluster, the applications frequently rely on publicly available container images, such as those for popular software like Jenkins or PostgreSQL. Since these images are hosted in external repositories beyond our control, there’s a chance that the repository owner could delete an image while our pods are still using it. If a node rotation occurs, the locally cached copies would be lost, and Kubernetes wouldn’t be able to re-download the images, preventing the applications from being re-provisioned.
- So, to address this, we need a controller that monitors the applications and "caches" the images by uploading them to our own registry. It should then update the applications to use these copies instead.

## Use

### Locally running the manager steps
- clone this repo "https://github.com/BadolJnU/Image_Controller_Kubernet.git"
- open the repo locally
- run `make`
- run `./bin/manager`
- open another terminal and go to samples: `cd config/samples`
- apply docker credential secret & deployment: 
    - give <base64 encoded username:password of your docker registry> in the `auth:` field of the docker-cred-k8s-secret 
    - run `kubectl apply -f docker-cred-secret.yaml`
    - run `kubectl apply -f sample-deployment.yaml`
- check in the sample deployment image, it will get cloned & pushed to your given docker registry and re-use in the deployment

### InCluster manager running steps
- `export IMG="<your_registry>/<controller_image_name>:<tag>"`
- `make docker-build`
- `make docker-push` (Note: for docker push you need to login in your dockerhub from the current terminal by `docker login`)
- `make deploy`
- verify the deployment by: `kubectl get all -n image-clone-controller-system`  
- open another terminal and go to samples: `cd config/samples`
- apply docker cred secret & sample deployment:
    - give <base64 encoded username:password of your docker registry> in the `auth:` field of the docker-cred-k8s-secret
    - run `kubectl apply -f docker-cred-secret.yaml`
    - run `kubectl apply -f sample-deployment.yaml`
- check in the sample deployment image, it will get cloned & pushed to your given docker registry and re-use in the deployment
- undeploy by: `make undeploy`

## e2e testing steps
- Added e2e test for deployment controller, similarly will add for DaemonSet controller
- For using Deployment controller test follow below steps:
  - run the controller (either locally or incluster running the manager)
  - in another terminal go to project's : `cd tests/e2e`
  - in the `tests/e2e/framework/docker-cred-secret.go` file provide your dockerhub "username:password" in the "auth" field
  - run `ginkgo run --which-controller=<controller_name> --registry=<your_dockerhub_username>`
  - ex: `ginkgo run -- --which-controller=deployment --registry=shahincsejnu`
  - Note: make sure you sync the namespace, registry name among test files & controllers
  
## Disclaimer
- It's a hobby project, not a production grade

## What's Next?
- make this controller code more generic 
- make helm chart of this operator

## Resources:
- https://book.kubebuilder.io/quick-start.html
- https://godoc.org/github.com/google/go-containerregistry/pkg/v1/remote  
- https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
- https://github.com/google/go-containerregistry/blob/main/pkg/authn/k8schain/README.md
- https://github.com/google/go-containerregistry/blob/main/cmd/crane/recipes.md



