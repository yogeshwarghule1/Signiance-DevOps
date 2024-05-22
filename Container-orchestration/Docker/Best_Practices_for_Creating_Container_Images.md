# Best Practices for Creating Container Images

Imagine running hundreds of microservices in production without knowing the dos and don'ts around creating container images. It’d be chaotic. You don’t want to spend hours building your container images and also you would not want to deploy images with vulnerabilities.

Containerization has become a cornerstone of modern software development and deployment, offering numerous benefits in terms of scalability, consistency, portability, and efficiency. One of the key elements in the container ecosystem is the creation of optimized and secure container images. In this blog, we will delve into the best practices for creating container images to help you build lightweight, reliable, and most importantly, secure images to make sure that your image is production-ready.

## Hacks for Optimizing Container Images

Creating a container image and optimizing it are two sides of a coin. For container creation, there are different ways which might involve writing a declarative Dockerfile or maybe using cloud-native buildpacks depending upon the use-cases. When it comes to optimization, it's true for all types of images being created. Here are some of the methods that can be used to optimize your container image and ensure the uptime of your applications.

### 1. Docker Hub Image Pull Rate Limit

In cases when you are dealing at scale, for instance - running a daemonset that is pulling its docker image from Docker Hub on a 100-node cluster at a time. In such scenarios, you’ll frequently face issues of rate limit. For example: “You have reached your pull rate limit.” How do we deal with it?

- **Look for an alternative container registry** where your helm chart’s public image could be available, for instance, QUAY or GHCR. In these container registries, there aren't any image pull limits. [Recommended]
- **Keep the image in your organization’s container registry** like ECR, ACR, etc. Make sure that the node has appropriate permissions for the pod to inherit. There would be an overhead to keep updating the image in case you are using your private registries for public helm charts.

**Note:** In the case of Docker Hub, 100 pull requests per 6 hours for anonymous users on a free plan. Enforced based on your IP Address. And 200 pull requests per 6 hours for authenticated users on a free plan.

### 2. Use Build Contexts

Avoid building your docker images like this - `docker build -t <tag> .`, unless you want to set the context as the current working directory. Here, the period (.) means context as the current working directory.

Let’s say you have two directories: `app` and `consumer` having the app’s code and consumer’s code respectively within the same repository then your build context should be `app/` and `consumer/` respectively. This would be helpful in scenarios where you are using `COPY . .` in Dockerfile as you will only be copying content from a specified folder instead of a complete repository and hence it’ll decrease time.

### 3. Multi-architecture Builds

If you have multi-architecture nodes in your cluster, i.e., ARM-based and AMD-based machines, then you will need to set the `--platform` flag to specify the target platform. By default, you can only build for a single platform at a time.

### 4. Implement Image Caching

A container image has multiple different layers. Imagine the size of the container image is in Gigabytes. Every time you build an image, it will start from the initial layers and build each and every layer. Those layers come from the steps given in your Dockerfile or the steps involved in building the image. Once you build your image, in subsequent builds, it will again create all those layers which will increase the time to build. To optimize the build time, it is highly recommended to implement a caching mechanism so that next time when a new image will be built, it will leverage the cache of the previous build for all those steps which were not changed and using the existing cache, build time will be reduced.

### 5. Golden Image Creation

It is one of the widely recommended practices to have golden images for your application. If all of your applications require a certain package, you don’t want to install it at run time every time. Instead, you can include it in the golden image and use that image as a base image for your container images. It’ll save a lot of time.

## Techniques for Optimizing Container Images

Now let's discuss some of the common techniques that can help you optimize your container images.

### 1. Multi-Stage Builds

Multi-stage builds enable you to create a final image that only includes the runtime dependencies and necessary artifacts. Unnecessary build tools, libraries, and intermediate files are discarded in subsequent stages, resulting in smaller image sizes. This reduction in size leads to faster image pulls and deployments, optimizing resource utilization. For applications such as Java which uses maven/gradle files, a multi-stage Dockerfile can be helpful.

### 2. Minimize Layers

Concatenate `RUN` commands to make your Dockerfile more readable and create fewer layers. Fewer layers mean a smaller container image. Each `RUN` statement in the Dockerfile creates a layer that gets cached. Concatenating reduces the number of layers.

### 3. Use Lightweight Images

Always choose the smallest base images that do not contain the complete or full-blown OS with system utilities installed. You can install the specific tools and utilities needed for your application in the Dockerfile build. This will reduce possible vulnerabilities and the attack surface of your image.

## Creating a Secure Container Image

Making sure the container image is secure and ready to be used in production is another important aspect and as a part of DevSecOps practices. Here are some of the practices that can be incorporated for building a secure image.

### 1. Environment Variables

Never inject environment variables within your container image. Embedding env variables within your container image means you are hardcoding them within the image. This way the image would not be generic to use across various microservices. So always use configMaps and Secrets to store env variables and credentials.

### 2. Vulnerability Scanning

Never deploy images with critical CVEs into production as this way you would be an easy target for hackers to hack into your application. Make sure to integrate vulnerability scanning tools in your pipeline so that you ship an application without vulnerabilities. On top of vulnerability scanning, having governance policies such as automatically blocking the image with critical CVEs can help you strengthen the pipeline.

### 3. Non-root User

By default, Docker containers run as the root user, which can pose security risks if the container gets compromised. In such cases, the hacker would have root access to your host and could bring the entire node down. So always ensure that your images run as non-root by defining `USER` in your Dockerfile.

### 4. Avoid using `<image>:latest`

The latest image tag for any public repository might bring in unexpected bugs. So it’s always recommended to specify a specific version instead of the latest.

## Conclusion

Creating high-quality container images is a crucial aspect of modern application development and deployment. By following these best practices, you can produce container images that are optimized, secure, and well-suited for deployment in various environments. Well-crafted images contribute to faster deployment, improved security, better resource utilization, and enhanced application performance, setting the foundation for successful container-based workflows.

**Tags:** docker, kubernetes
<br>
**~Kamran**
