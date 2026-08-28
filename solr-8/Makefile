.PHONY: all help build push check-schema
SHELL := /bin/bash
CKAN_VERSION ?= 2.10
CKAN_BRANCH ?= dev-v$(CKAN_VERSION)
SOLR_VERSION ?= 8
IMAGE_NS ?= mjanez/ckan-solr
TAG_NAME="$(IMAGE_NS):$(CKAN_VERSION)-solr8"
TAG_NAME_SPATIAL="$(IMAGE_NS):$(CKAN_VERSION)-solr8-spatial"

all: help
help:
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'

build: ## Build Solr 8 images, `make build CKAN_VERSION=2.10` (default is 2.10)
	docker build -t $(TAG_NAME) \
		--build-arg CKAN_BRANCH=$(CKAN_BRANCH) \
		--build-arg SOLR_VERSION=$(SOLR_VERSION) .
	docker build -t $(TAG_NAME_SPATIAL) \
		-f Dockerfile.spatial \
		--build-arg CKAN_BRANCH=$(CKAN_BRANCH) \
		--build-arg SOLR_VERSION=$(SOLR_VERSION) .

push: ## Push images, `make push CKAN_VERSION=2.10`
	docker push $(TAG_NAME)
	docker push $(TAG_NAME_SPATIAL)

check-schema: ## Apply schema/ extras onto the CKAN schema (no Docker)
	@curl -fsSL "https://raw.githubusercontent.com/ckan/ckan/$(CKAN_BRANCH)/ckan/config/solr/schema.xml" -o /tmp/ckan-schema.xml
	@cat schema/spatial-fields.xml schema/dcat-ap-fields.xml schema/dcat-ap-es-fields.xml > /tmp/extra-fields.xml
	@sed "/<types>/r schema/spatial-types.xml" /tmp/ckan-schema.xml | sed "/<fields>/r /tmp/extra-fields.xml" > /tmp/ckan-managed-schema
	@echo "schema name:" && grep -E '<schema name=' /tmp/ckan-managed-schema
	@echo "injected:" && grep -E 'name="(location_rpt|spatial_geom|hvd_category|endpoint_url|dcat_type|theme_es|dataset_scope)"' /tmp/ckan-managed-schema
	@echo "written to /tmp/ckan-managed-schema"
