This is a hello-world HTTP server that prints hostname and arch information. 

The go code is adapted from
https://github.com/GoogleCloudPlatform/kubernetes-engine-samples/blob/master/hello-app/main.go.

# Image

The hello world image is available publicly at docker.io/congliu01/hello-world-go:latest

Run the image: `docker run docker.io/congliu01/hello-world-go:latest`

# Build

```bash
# Define some variables
export REGISTRY="docker.io/congliu01"
export IMAGE_NAME="hello-world-go"
export TAG="latest"

# Install QEMU packages
docker run --rm --privileged docker/binfmt:66f9012c56a8316f9244ffd7622d7c21c1f6f28d

# Create a multi-arch builder (default builder doesn't support multi-arch)
docker buildx create --use --name mybuilder
```

## Option 1 QEMU

```bash
MULTI_ARCH_IMAGE="${REGISTRY}/${IMAGE_NAME}"
docker buildx build . -t "${MULTI_ARCH_IMAGE}:${TAG}" --platform linux/amd64,linux/arm64 --push
```

To test the image,

```bash
docker run  "${MULTI_ARCH_IMAGE}:${TAG}"
```

## Option 2 Cross Compile

### Option 2.1 Build and Push with Buildx

```bash
MULTI_ARCH_IMAGE="${REGISTRY}/${IMAGE_NAME}-cross"

docker buildx build . -f ./Dockerfile.cross -t "${MULTI_ARCH_IMAGE}:${TAG}" --platform linux/amd64,linux/arm64 --push
```

To test the image,

```bash
docker run  "${MULTI_ARCH_IMAGE}:${TAG}"
```

### Option 2.2 Docker Manifest List

#### Option 2.2.1 With Buildx

```bash
MULTI_ARCH_IMAGE="${REGISTRY}/${IMAGE_NAME}-cross2"
AMD64_IMAGE="${MULTI_ARCH_IMAGE}-linux-amd64"
ARM64_IMAGE="${MULTI_ARCH_IMAGE}-linux-arm64"

# build and push images for each arch
# use --load flag to load the docker image locally before pushing
docker buildx build . -f ./Dockerfile.linux-amd64 -t "${AMD64_IMAGE}:${TAG}" --load
docker push "${AMD64_IMAGE}:${TAG}" 
docker buildx build . -f ./Dockerfile.linux-arm64 -t "${ARM64_IMAGE}:${TAG}" --load
docker push "${ARM64_IMAGE}:${TAG}" 

# create a manifest list (multi-arch image) and add each arch to it
docker manifest create --amend "${MULTI_ARCH_IMAGE}:${TAG}" "${AMD64_IMAGE}:${TAG}" "${ARM64_IMAGE}:${TAG}"  
# annotate the images with their corresponding architecture
docker manifest annotate --os=linux --arch=amd64 "${MULTI_ARCH_IMAGE}:${TAG}" "${AMD64_IMAGE}:${TAG}"
docker manifest annotate --os=linux --arch=arm64 "${MULTI_ARCH_IMAGE}:${TAG}" "${ARM64_IMAGE}:${TAG}"  
# push the manifest list
docker manifest push "${MULTI_ARCH_IMAGE}:${TAG}"
```

To test the image,

```bash
docker run  "${MULTI_ARCH_IMAGE}:${TAG}"
```

#### Option 2.2.2 With Normal Docker Build

```bash
MULTI_ARCH_IMAGE="${REGISTRY}/${IMAGE_NAME}-cross3"
AMD64_IMAGE="${MULTI_ARCH_IMAGE}-linux-amd64"
ARM64_IMAGE="${MULTI_ARCH_IMAGE}-linux-arm64"

# build and push images for each arch
docker build . -f ./Dockerfile.linux-amd64 -t "${AMD64_IMAGE}:${TAG}"
docker push "${AMD64_IMAGE}:${TAG}" 
docker build . -f ./Dockerfile.linux-arm64 -t "${ARM64_IMAGE}:${TAG}"
docker push "${ARM64_IMAGE}:${TAG}" 

# create a manifest list (multi-arch image) and add each arch to it
docker manifest create --amend "${MULTI_ARCH_IMAGE}:${TAG}" "${AMD64_IMAGE}:${TAG}" "${ARM64_IMAGE}:${TAG}"  
# annotate the images with their corresponding architecture
docker manifest annotate --os=linux --arch=amd64 "${MULTI_ARCH_IMAGE}:${TAG}" "${AMD64_IMAGE}:${TAG}"
docker manifest annotate --os=linux --arch=arm64 "${MULTI_ARCH_IMAGE}:${TAG}" "${ARM64_IMAGE}:${TAG}"  
# push the manifest list
docker manifest push "${MULTI_ARCH_IMAGE}:${TAG}"
```

To test the image,

```bash
docker run  "${MULTI_ARCH_IMAGE}:${TAG}"
```