REPO := acrveceusgloshared001.azurecr.io
DOCKERFILE := dockerfile
PLATFORM := linux/amd64
ORGANIZATION := cezzis
IMAGE_NAME := k3s
TAG := v1.31.5-k3s1.12.2.2-cuda12.2.2-base-ubuntu22.04-v1

# Default target
all: build-k3d push-k3d

build-k3d:
	@docker build --platform=${PLATFORM} -t ${REPO}/${ORGANIZATION}/${IMAGE_NAME}:${TAG} -f ${DOCKERFILE} .

push-k3d:
	@az acr login -n acrveceusgloshared001
	@docker push ${REPO}/${ORGANIZATION}/${IMAGE_NAME}:${TAG}

.PHONY: all build-k3d push-k3d