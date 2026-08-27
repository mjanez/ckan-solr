# ckan-solr

Pre-configured Solr Docker images for [`ckan-docker *spatial`](https://github.com/mjanez/ckan-docker) and [`ckanext-schemingdcat`](https://github.com/mjanez/ckanext-schemingdcat).

Published to **GHCR** as `ghcr.io/mjanez/ckan-solr`. These are **not** the official `ckan/ckan-solr` images: the `-spatial` tags add JTS + RPT geo fields **and** explicit DCAT/GeoDCAT-AP facet fields.

Images are built from [upstream Solr](https://github.com/apache/solr-docker#readme). Re-pull to pick up Solr patch releases baked at build time. Solr **9.9** is the pinned minor for Solr 9 tags.

```bash
docker run --name ckan-solr -p 8983:8983 -d ghcr.io/mjanez/ckan-solr:2.11-solr9-spatial
```

CKAN `solr_url`: **http://localhost:8983/solr/ckan**

## Tags

| CKAN | Solr | Image | Notes |
| --- | --- | --- | --- |
| **2.11** | **9.9** | `ghcr.io/mjanez/ckan-solr:2.11-solr9-spatial` | Recommended. Also tagged `2.11-solr9.9-spatial` |
| 2.11 | 9.9 | `ghcr.io/mjanez/ckan-solr:2.11-solr9` | Stock CKAN schema only |
| 2.10 | 9.9 | `ghcr.io/mjanez/ckan-solr:2.10-solr9-spatial` | Also tagged `2.10-solr9.9-spatial` |
| 2.10 | 9.9 | `ghcr.io/mjanez/ckan-solr:2.10-solr9` | Stock CKAN schema only |
| 2.10 | 8 | `ghcr.io/mjanez/ckan-solr:2.10-solr8-spatial` | Legacy Solr 8 |
| 2.10 | 8 | `ghcr.io/mjanez/ckan-solr:2.10-solr8` | Legacy Solr 8, stock schema |
| 2.9 | 9.9 | `ghcr.io/mjanez/ckan-solr:2.9-solr9-spatial` | Requires CKAN ≥ 2.9.5 |
| 2.9 | 8 | `ghcr.io/mjanez/ckan-solr:2.9-solr8-spatial` | Requires CKAN ≥ 2.9.5 |

The schema `name` attribute is taken from the matching CKAN branch (`dev-v2.10` -> `ckan-2.10`, `dev-v2.11` -> `ckan-2.11`). CKAN will not start if it does not match the series.

The following tags are no longer supported:

| CKAN Version | Solr version | Docker tag | Legacy Docker tags | Notes |
| --- | --- | --- | --- | --- |
| 2.9 | 9.9 | `ghcr.io/mjanez/ckan-solr:2.9-solr9-spatial` | Requires CKAN ≥ 2.9.5 |
| 2.9 | 8 | `ghcr.io/mjanez/ckan-solr:2.9-solr8-spatial` | Requires CKAN ≥ 2.9.5 |

## What `-spatial` adds vs upstream `ckan/ckan-solr`

Same as official spatial images:

- JTS Core **1.19.0**
- `location_rpt` (`SpatialRecursivePrefixTreeFieldType`, JTS, `repairBuffer0`)
- `spatial_geom`, `bbox_area`, `minx`, `maxx`, `miny`, `maxy`

Plus DCAT facet fields (`string` + `docValues` + `multiValued`), which CKAN `extras_*` cannot facet correctly. They live under `schema/` as separate fragments so a portal can drop a profile without editing the spatial block:

| File | Fields |
| --- | --- |
| `spatial-types.xml` / `spatial-fields.xml` | `location_rpt`, `spatial_geom`, bbox floats, `spatial_uri` |
| `dcat-ap-fields.xml` | `tag_uri`, `alternate_identifier`, `theme`, `theme_eu`, `language`, `dcat_type`, `conforms_to`, `applicable_legislation`, `hvd_category`, `publisher_name`, `publisher_type`, `frequency`, `endpoint_url`, `serves_dataset`, `reference`, `is_referenced_by`, `resource_relation`, `documentation`, `metadata_profile`, `lineage_source`, `lineage_process_steps` |
| `dcat-ap-es-fields.xml` | `theme_es`, `dataset_scope` |

`Dockerfile.spatial` concatenates the field files after `<fields>`. The stock `Dockerfile` does not.

Use `ckanext.spatial.search_backend = solr-spatial-field` with the spatial tags.

## Building locally

```bash
cd solr-9
make build                          # CKAN 2.11 / Solr 9.9
make build CKAN_VERSION=2.10        # CKAN 2.10 / Solr 9.9

cd ../solr-8
make build CKAN_VERSION=2.10
```

## Custom config files

1. `docker run --name ckan-solr -p 8983:8983 -d ghcr.io/mjanez/ckan-solr:2.11-solr9-spatial`
2. `docker cp ckan-solr:/opt/solr/server/solr/ckan/conf ./my_conf`
3. `docker stop ckan-solr`
4. `docker run -p 8983:8983 --mount type=bind,source="$(pwd)"/my_conf,target=/opt/solr/server/solr/ckan/conf -d ghcr.io/mjanez/ckan-solr:2.11-solr9-spatial`
5. Reload the core: http://localhost:8983/solr/#/~cores/
