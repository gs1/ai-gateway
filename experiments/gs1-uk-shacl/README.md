# GS1 UK SHACL validation profile (proposed v1.0)

A SHACL profile for GS1 product master data published as JSON-LD.

- **Origin:** GS1 UK product-data validation experiment. GS1 France reviewed the
  profile and contributed technical corrections.
- **Purpose:** check that product data uses GS1 Web Vocabulary classes,
  properties, datatypes and code-list values correctly.
- **Proposed version:** v1.0. This is a starting point for discussion, not a
  finished profile.
- **Reference vocabulary:** GS1 Web Vocabulary 1.18.0, pinned (see
  [Dependency](#dependency-gs1-web-vocabulary-1180)).

This profile is not an official GS1 standard. Conforming to it does not mean
the data is certified by GS1 or conforms to any GS1 standard.

## Contents

| Path | Description |
| --- | --- |
| `shapes/master-product-data-shacl.ttl` | The SHACL profile: 88 node shapes and 372 property shapes, 4,293 triples |

The ten example product records used for the review are **not included yet**.
They are real product records from GS1 UK member companies. They will be added
to this directory once public redistribution of their content has been
confirmed.

## Dependency: GS1 Web Vocabulary 1.18.0

Ontology-aware validation requires this exact reference file. It is not
included in this repository.

| | |
| --- | --- |
| URL | <https://ref.gs1.org/voc/data/1.18.0/gs1Voc.ttl> |
| Version | 1.18.0 (`owl:versionInfo "1.18"`, modified 2026-06-01) |
| SHA-256 | `b6039ed1b3229853a0aed1eb8568527f2d393925fd163d6fce5fddefe7ea1aa7` |
| Licence | © GS1 AISBL, Apache License 2.0, with the GS1 IP Policy notice in the file |

The profile was reviewed against 1.18.0 only. Do not substitute a later
release, such as 1.19.0, without reviewing the profile again. Class and
property declarations, datatype ranges and code lists can change between
releases.

## Validation configuration

- **Load the vocabulary as an ontology graph.** Use pySHACL `-e`. Do not
  merge the vocabulary into the data graph or add `owl:imports`.
  `GS1ClassValidationShape` uses a SPARQL constraint that reports any
  `https://ref.gs1.org/voc/` class not declared as an `owl:Class`. If the
  vocabulary is not loaded, every GS1 class is reported as invalid.
- **Inference:** `none`. The 10/10 result below applies to this setting only.
  It was not obtained with RDFS inference when `GS1ClassValidationShape` is
  active.
- **Language tags:** text properties whose 1.18.0 range is `rdf:langString`
  must have a language tag, for example
  `{"@value": "...", "@language": "en"}`. Untagged strings are rejected.

## Results

### Local evidence (not reproducible from this repository yet)

GS1 France validated the profile locally against ten JSON-LD product records
supplied by GS1 UK, using the configuration above. The tools were Python
3.14.7, RDFLib 7.6.0 and pySHACL 0.40.2.

- **10 of 10 records conform, with 0 validation results.**
- Two controls confirmed that the setup is active:
  - Without the vocabulary, a conforming record fails with 14 results.
  - A record with an untagged `productName` fails with 1 result.

These records are not published here, so this result cannot yet be checked
from this repository. It will become reproducible once the examples are added.

### Reproducible from the published files

Anyone can reproduce the following with the files in this directory and the
pinned vocabulary:

- the SHACL file parses (4,293 triples, 88 node shapes, 372 property shapes);
- the vocabulary checksum matches the value above;
- validation of your own GS1 JSON-LD data with the configuration above.

## How to validate

Run these commands from this directory:

```sh
python -m venv .venv
. .venv/bin/activate            # Windows: .venv\Scripts\activate
pip install pyshacl==0.40.2

# GS1 Web Vocabulary 1.18.0. Do not commit this file.
curl -L -o gs1Voc-1.18.0.ttl https://ref.gs1.org/voc/data/1.18.0/gs1Voc.ttl
sha256sum gs1Voc-1.18.0.ttl
# expected: b6039ed1b3229853a0aed1eb8568527f2d393925fd163d6fce5fddefe7ea1aa7

pyshacl -s shapes/master-product-data-shacl.ttl -sf turtle \
        -e gs1Voc-1.18.0.ttl -ef turtle \
        -i none -a -df json-ld -f human path/to/product.json
```

Replace `path/to/product.json` with your own file. A conforming file prints
`Conforms: True`. A non-conforming file makes `pyshacl` exit with code 1 and
list the violations.

## Known limitations

The profile checks the values that are present. It does not check that a
record is complete. The following points are open for discussion:

- **Identifiers:** GTIN format and check digit are not checked, so a GTIN of
  `"123"` is accepted.
- **Digital Link URIs:**
  - The profile does not check that a product's `@id` matches its GTIN.
  - In the reviewed records, the organisation IRI uses the GTIN path (`/01/`)
    where a GLN (`/417/` or `/414/`) would be expected.
- **Links:**
  - `pip`, `hasRetailers`, `relatedImage`, `relatedVideo` and
    `certificationInfo` are not validated.
  - The predicate for each link `@description` is still open, as is the choice
    between IRIs and `xsd:anyURI` literals.
  - In the reviewed records, seven `@description` values do not produce RDF
    triples.
- **Packaging:** it is not yet decided which `PackagingDetails` node should
  carry each `packagingMaterial`. In the reviewed records, four product-level
  `packagingMaterial` blocks do not produce RDF triples.
- **Organisation, address and contact data:** not validated, because
  `OrganizationShape` has no property shapes. In the reviewed records, 47 of
  458 GS1 statements are not reached by any shape.
- **Code lists:**
  - Some lists are subsets of 1.18.0. For example, the allergen list lacks 173
    codes.
  - Deprecated codes, such as `MANUFACTURER_OF_GOODS`, are still accepted.
  - 61 shapes are defined but never applied.
- **Modelling:**
  - The `sh:or` in `ProductShape` does not distinguish product categories.
  - Two shapes reject valid values: `PreservationTechniqueShape` and
    `AwardPrizeCountryCodeShape`.
  - The profile has no cardinality constraints.

## Licence

The SHACL profile and this README are contributed under this repository's
[Apache License 2.0](../../LICENSE). The GS1 Web Vocabulary is © GS1 AISBL and
is not redistributed here.
