MAKEFLAGS   += --warn-undefined-variables
SHELL       := /bin/bash
.SHELLFLAGS := -euo pipefail -c

IMG_TAG     = test
IMG         = local/external-secrets-e2e:$(IMG_TAG)
KIND_IMG    = "kindest/node:v1.21.1@sha256:69860bda5563ac81e3c0057d654b5253219618a22ec3a346306239bba8cfa1a6"
BUILD_ARGS  ?=
export FOCUS := $(FOCUS)

start-kind: ## Start kind cluster
	kind create cluster \
	  --name external-secrets \
	  --config kind.yaml \
	  --retain \
	  --image "$(KIND_IMG)"

test: e2e-image ## Run e2e tests against current kube context
	$(MAKE) -C ../ docker.build \
		IMAGE_REGISTRY=local/external-secrets \
		VERSION=$(IMG_TAG) \
		ARCH=amd64 \
		BUILD_ARGS="${BUILD_ARGS} --build-arg TARGETARCH=amd64 --build-arg TARGETOS=linux"
	kind load docker-image --name="external-secrets" local/external-secrets:$(IMG_TAG)
	kind load docker-image --name="external-secrets" $(IMG)
	./run.sh

e2e-bin:
	CGO_ENABLED=0 go run github.com/onsi/ginkgo/ginkgo build .

e2e-image: e2e-bin
	-rm -rf ./k8s/deploy
	mkdir -p k8s
	$(MAKE) -C ../ helm.generate
	cp -r ../deploy ./k8s
	docker build $(BUILD_ARGS) -t $(IMG) .

stop-kind: ## Stop kind cluster
	kind delete cluster \
		--name external-secrets \

help: ## displays this help message
	@awk 'BEGIN {FS = ":.*?## "} /^[a-zA-Z_\/-]+:.*?## / {printf "\033[34m%-18s\033[0m %s\n", $$1, $$2}' $(MAKEFILE_LIST) | \
		sort | \
		grep -v '#'
