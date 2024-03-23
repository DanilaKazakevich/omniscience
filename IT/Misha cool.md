```sh
.PHONY: all
.DEFAULT_GOAL := help

# ----------------------------------------------------------------------------
# Local Variables
#
# ============================================================================

PRIVATE_DOCKER_REGISTRY=saritasallc
TELEPORT_VERSION = 12 13 14 15

help:
	@grep -E '^[\.0-9a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "⚡ \033[34m%-30s\033[0m %s\n", $$1, $$2}'

# ----------------------------------------------------------------------------
# ECR/Docker Helper Commands
#
# ============================================================================

private_docker_login: ## Login to Private Docker Hub Registry
	pass ${PRIVATE_DOCKER_REGISTRY}/docker/password | docker login -u ${PRIVATE_DOCKER_REGISTRY} docker.io --password-stdin

# ----------------------------------------------------------------------------
# Main command for collecting images
#
# ============================================================================

collect_all: private_docker_login $(TELEPORT_VERSION)

$(TELEPORT_VERSION): 
	$(call collect,teleport-distroless,$@)
	$(call collect,teleport-operator,$@)

# ----------------------------------------------------------------------------
# Build functions
#
# ============================================================================

# Fetch image from ECR Registry and retag it to PRIVATE_DOCKER_REGISTRY
# $1 - image
# $2 - version
define collect
	@docker pull public.ecr.aws/gravitational/$1:$2
	@docker tag public.ecr.aws/gravitational/$1:$2 ${PRIVATE_DOCKER_REGISTRY}/$1:$2
	@docker push ${PRIVATE_DOCKER_REGISTRY}/$1:$2
	$(call ok, "Successfully mirrored $1 $2")
endef
```

