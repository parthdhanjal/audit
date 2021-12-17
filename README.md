[![Go Report Card](https://goreportcard.com/badge/github.com/camilamacedo86/audit)](https://goreportcard.com/report/github.com/camilamacedo86/audit)
[![Coverage Status](https://coveralls.io/repos/github/github.com/operator-framework/audit/badge.svg?branch=main)](https://coveralls.io/github/camilamacedo86/audit?branch=main)

---
# Audit

**IMPORTANT** This project is a POC. Before running the reports, ensure that you have its latest version by running `git pull` and `git status` if your installing from the source.

## Overview

The audit is an **experimental** analytic tool which uses the Operator Framework solutions. Its purpose is to obtain and report and aggregate data provided by checks and analyses done in the operator bundles, packages and channels from an index catalog image.

Note that the latest version of the reports generated for all images can be checked in [testdata/report](testdata/reports). The file names are create by using the kind/type of the report, image name and date. (E.g. `testdata/report/bundles_quay.io_operatorhubio_catalog_latest_2021-04-22.xlsx`).

For further information about its motivation see the [EP Audit command operation][audit-ep]. 

## Pre-requirements

- go 1.16 (only if you would like to install from the source)
- docker or podman
- access to the registry where the index catalog and operator bundle images are distribute
- access to a Kubernetes cluster
- [operator-sdk][operator-sdk] installed >= `1.5.0

**NOTE** that you can run the reports without SDK and the cluster running with by using the flag `--disable-scorecard`. That is only required for the scorecard results.  

## Install binary:

Check the release binaries provided in the [release page](https://github.com/operator-framework/audit/releases).

## Install from the source

To get the project and install the binary:

```sh
$ git clone git@github.com:operator-framework/audit.git
$ cd audit
$ make install
```

## Usage

### Ensure that you have access to pull the images

You may first need to run `docker login` or `podman login` to have access to the images.

### Podman

Per default the audit commands use docker for dealing with container images. If you wish to use podman instead

- either set the environment variable `export CONTAINER_ENGINE=podman` beforehand.
- or add `--container-engine=podman` to each command

```sh
export CONTAINER_ENGINE=podman
```

### Generating the reports

Now, you can audit all operator bundles of an image catalog with: 

```sh 
audit-tool index bundles --index-image=registry.redhat.io/redhat/redhat-operator-index:v4.7 --head-only 
```


### Options

Use the `--help` flag to check the options and the further information about its commands. Following an example:

```sh
$ audit-tool index bundles --help
Provides reports with the details of all bundles operators ship in the index image informed according to the criteria defined via the flags.

 **When this report is useful?** 

This report is useful when is required to check the operator bundles details.

Usage:
  audit-tool bundles [flags]

Flags:
...
```

### Filtering results by names

See that you can use the `--filter` --flag to filter the results by the package name:

```sh
audit-tool index [bundles] --index-image=registry.redhat.io/redhat/redhat-operator-index:v4.5 --filter="mypackagename"
```

### Option to run in dedicated environments

Use the flag `--server-mode` to generate the reports in dedicated environments. By using this flag option the images
which are downloaded will not be removed, allowing the reports to be generated faster after the first execution.

Also, ensure that you have enough space to store all images. Note that the default behavior is to remove them, when this option is not used.  

## Reports

| Report Type | Command | Description |
| ------ | ----- |  ------ |
| bundles | `audit index bundle --index-image [OPTIONS]` | Audit all Bundles |

## Testdata

The samples in `testdata/samples` which are generated by running `make generate-samples`. Also, to run `make generate-testdata` to re-generate all reports in the testdata.

## Custom Dashboards

In order to address specific needs, audit has been used to generate custom dashboards. The dashboards are generated using the JSON results provided by the audit index reports, e.g.:

```sh
audit-tool dashboard deprecate-apis --file=testdata/report/bundles_quay.io_operatorhubio_catalog_latest_2021-04-22.json 
```

## Index page

The `index.html` page is generated via `make generate-index`. It will aggregate in its results all dashboards found per image which are available in the testdata. To check it, see https://operator-framework.github.io/audit/ . 

## FAQ

### How Audit works?

Following the steps performed by Audit. 

- Extract the database from the image informed
- Perform SQL queries to obtain the data from the index db
- Download and extract all bundles files by using the operator bundle path which is stored in the index db  
- Get the required data for the report from the operator bundle manifest files 
- Use the [operator-framework/api][of-api] to execute the bundle validator checks
- Use SDK tool to execute the Scorecard bundle checks
- Output a report providing the information obtained and processed. 

For some detailed information about its implementation check [here](docs/steps.md).

### What means UNKNOWN ?

The UNKNOWN status means that was not possible gathering the information, usually because was not possible to download the operator bundle to check it.

### What means NOT USED ?

If you see a column with this information than that means that the specific criteria is not useful or applied to none operator bundle of a package or the specific bundle itself.

### What are the images used to generate the full reports?

- OCP images: See [Understanding Operator catalogs](https://github.com/openshift/openshift-docs/blob/master/modules/olm-understanding-operator-catalog-images.adoc#understanding-operator-catalogs)
- Community operator image (`quay.io/operatorhubio/catalog:latest`): Its source is from [upstream-community-operators](https://github.com/operator-framework/community-operators/tree/master/upstream-community-operators)

[of-api]: https://github.com/operator-framework/api
[scorecard-config]: https://github.com/operator-framework/operator-sdk/blob/v1.5.0/testdata/go/v3/memcached-operator/bundle/tests/scorecard/config.yaml
[operator-sdk]: https://github.com/operator-framework/operator-sdk
[audit-ep]: https://github.com/operator-framework/enhancements/blob/master/enhancements/audit-command.md

### Release Process

Only creates and push a new tag then, the github actions will build and add the artefacts in the release page. 
