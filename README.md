# sml-tpcds-common

An AtScale SML package with shared dimensions, metrics, and calculations for TPC-DS semantic models.

SML (Semantic Modeling Language) is the YAML format in which AtScale stores a semantic model as code. An SML package is a repository that other SML repositories import, so that one definition serves several models. This repository has no catalog and no model of its own. The consuming repository [sml-tpcds-snowflake](https://github.com/muellermarco/sml-tpcds-snowflake) imports it. The main readers are AtScale modelers who change shared TPC-DS objects or build a new TPC-DS model on them.

## Technologies

- SML version 1.5 (the version of the consuming catalog)
- AtScale 2026.9.2
- `sml-cli` 2025.12.0

## Contents

| Object | Count | Names |
|---|---|---|
| Dimensions | 3 | Customer Dimension, Date Dimension, Product Dimension |
| Metrics | 11 | Store and web measures for net paid, net profit, and quantity, and the web sales row counter (`metrics/`) |
| Calculations | 4 | Avg Store Unit Net Profit, Customer Count, Total Net Profit, Total Quantity Sold (`calculations/`) |

## What the consuming repository must define

The objects in this package refer to datasets and metrics by name, but the package does not define them. The consuming repository must define these objects with the same `unique_name`:

| Object type | Names | Used by |
|---|---|---|
| Dataset | `customer`, `customer_address` | Customer Dimension |
| Dataset | `date_dim` | Date Dimension |
| Dataset | `item` | Product Dimension |
| Dataset | `store_sales`, `web_sales` | The metrics |
| Metric | `m_SS_CUSTOMER_SK_count`, `m_WS_BILL_CUSTOMER_SK_count` | Customer Count |

If a name is missing in the consuming repository, `sml-cli validate` fails there.

## Use the package in a model repository

Add an entry to `package.yml` of the consuming repository. This is the entry in `sml-tpcds-snowflake`:

```yaml
version: 1
packages:
 - name: Common-Objects
   url: https://github.com/muellermarco/sml-tpcds-common.git
   branch: main
   version: commit:<sha>
```

Then, in the consuming repository:

1. Run `npx -y sml-cli@2025.12.0 install .`
   Result: `sml-cli` downloads this package at the pinned commit.
2. Run `npx -y sml-cli@2025.12.0 validate .`
   Result: the output contains `Validation SUCCESSFUL`.

## Change the package

Consumers pin this package to a commit, so a change here has no effect until a consumer moves its pin.

1. Change the objects on a branch and open a pull request.
2. Merge the pull request to `main`.
3. Copy the full commit SHA of the merge.
4. In each consuming repository, set `version: commit:<sha>` in `package.yml`.
5. In each consuming repository, run `sml-cli install` and `sml-cli validate`, then open a pull request.
   Result: the consumer's `validate` check tests the new package version.

This repository is not on the `atscale-cicd-pipeline`. It has no registry entry and no `validate` workflow of its own. The validate run of each consuming repository checks it. The AtScale instances have it registered as the repository connection `tpcds-common`, so that Design Center can resolve the package.

## License

MIT. See [LICENSE](LICENSE).
