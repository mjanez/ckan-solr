# Solr schema extras

Fragments injected into CKAN `schema.xml` at image build (`Dockerfile.spatial` concatenates the field files after `<fields>`).

| File | Scope |
|------|--------|
| `spatial-types.xml` | `location_rpt` (JTS / RPT). Required by ckanext-spatial `solr-spatial-field`. |
| `spatial-fields.xml` | Geo search: `spatial_geom`, bbox floats, `spatial_uri`. Same as official `ckan/ckan-solr:*-spatial` plus `spatial_uri`. |
| `dcat-ap-fields.xml` | DCAT-AP / DCAT-AP 3 / GeoDCAT-AP facets (`theme`, `theme_eu`, HVD, DataService, lineage, …). Lookup fields (`alternate_identifier`) live here too. |
| `dcat-ap-es-fields.xml` | DCAT-AP-ES: `theme_es`, `dataset_scope`. |

CKAN `extras_*` is `text` (tokenized) and cannot facet URI lists. These fields are `string` + `docValues` + `multiValued`.

The stock `Dockerfile` (no `-spatial` tag) does not apply these files.
