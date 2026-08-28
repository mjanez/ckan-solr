.PHONY: all help build push check-schema check-image
SHELL := /bin/bash
CKAN_VERSION ?= 2.11
CKAN_BRANCH ?= dev-v$(CKAN_VERSION)
SOLR_VERSION ?= 10.0
IMAGE_NS ?= mjanez/ckan-solr
TAG_NAME="$(IMAGE_NS):$(CKAN_VERSION)-solr10"
ALT_TAG_NAME="$(IMAGE_NS):$(CKAN_VERSION)-solr$(SOLR_VERSION)"
TAG_NAME_SPATIAL="$(IMAGE_NS):$(CKAN_VERSION)-solr10-spatial"
ALT_TAG_NAME_SPATIAL="$(IMAGE_NS):$(CKAN_VERSION)-solr$(SOLR_VERSION)-spatial"

all: help
help:
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'

build: ## Build Solr 10 images, `make build CKAN_VERSION=2.12` (default is 2.11)
	docker build -t $(TAG_NAME) -t $(ALT_TAG_NAME) \
		--build-arg CKAN_BRANCH=$(CKAN_BRANCH) \
		--build-arg SOLR_VERSION=$(SOLR_VERSION) .
	docker build -t $(TAG_NAME_SPATIAL) -t $(ALT_TAG_NAME_SPATIAL) \
		-f Dockerfile.spatial \
		--build-arg CKAN_BRANCH=$(CKAN_BRANCH) \
		--build-arg SOLR_VERSION=$(SOLR_VERSION) .

push: ## Push images, `make push CKAN_VERSION=2.12` (default is 2.11)
	docker push $(TAG_NAME)
	docker push $(ALT_TAG_NAME)
	docker push $(TAG_NAME_SPATIAL)
	docker push $(ALT_TAG_NAME_SPATIAL)

check-schema: ## Apply schema/ extras onto the CKAN schema (no Docker)
	@curl -fsSL "https://raw.githubusercontent.com/ckan/ckan/$(CKAN_BRANCH)/ckan/config/solr/schema.xml" -o /tmp/ckan-schema.xml
	@cat schema/spatial-fields.xml schema/dcat-ap-fields.xml schema/dcat-ap-es-fields.xml > /tmp/extra-fields.xml
	@sed "/<types>/r schema/spatial-types.xml" /tmp/ckan-schema.xml | sed "/<fields>/r /tmp/extra-fields.xml" > /tmp/ckan-managed-schema
	@echo "schema name:" && grep -E '<schema name=' /tmp/ckan-managed-schema
	@echo "injected:" && grep -E 'name="(location_rpt|spatial_geom|hvd_category|endpoint_url|dcat_type|theme_es|dataset_scope)"' /tmp/ckan-managed-schema
	@echo "written to /tmp/ckan-managed-schema"

check-image: ## Inspect managed-schema + Solr ping inside the spatial image
	docker run --rm --entrypoint cat $(TAG_NAME_SPATIAL) \
		/opt/solr/server/solr/configsets/ckan/conf/managed-schema > /tmp/ckan-managed-schema-image
	@echo "schema name:" && grep -E '<schema name=' /tmp/ckan-managed-schema-image
	@grep -q 'name="spatial_geom"' /tmp/ckan-managed-schema-image
	@grep -q 'name="hvd_category"' /tmp/ckan-managed-schema-image
	@grep -q 'name="dcat_type"' /tmp/ckan-managed-schema-image
	@grep -q 'name="theme_es"' /tmp/ckan-managed-schema-image
	@echo "JTS jar:" && docker run --rm --entrypoint ls $(TAG_NAME_SPATIAL) /opt/solr/server/solr-webapp/webapp/WEB-INF/lib | grep jts-core
	docker run --rm -d --name ckan-solr-check -p 8983:8983 $(TAG_NAME_SPATIAL)
	@for i in $$(seq 1 30); do curl -sf http://localhost:8983/solr/ckan/admin/ping && break; sleep 2; done
	@curl -sf "http://localhost:8983/solr/ckan/schema/fields?wt=json" | grep -E 'spatial_geom|hvd_category|dcat_type|theme_es'
	docker rm -f ckan-solr-check
	@echo "OK $(TAG_NAME_SPATIAL)"
