# PCF Tile Generator

This is a tile generation utility for PCF tiles. Tiles are the
installation package format used by Pivotal's Ops Manager to deploy add-on
software such as services and their brokers, buildpacks, or anything else
that needs to be installable in both public and private cloud deployments.

The current release of the tile generator supports tiles that have any
combination of the following package types:
- Cloud Foundry Applications
- Cloud Foundry Buildpacks
- Cloud Foundry Service Brokers (both inside and outside the Elastic Runtime)
- Docker images (both inside and outside the Elastic Runtime)

## How to Use

1. Create a repository for your tile (preferably git, but this is not required)
2. Initialize it as a tile repo using `tile init`
3. Configure the required properties as indicated by `tile status`
  - `tile set-icon <image>` (the icon image shown on the tile)
  - `tile set-label <label>` (the short label shown on the tile)
  - `tile set-description <description>` (the longer description of the tile's purpose)
4. Add each of the packages you want to install (current only buildpacks)
  - `tile add-buildpack <name> <zipfile> <rank>`
5. Build your tile using `tile build`

The generator will first create a BOSH release (in the `release` subdirectory),
then wrap that release into a Pivotal tile (in the `product` subdirectory).
If required for the installation, it will automatically pull down the latest
release version of the Cloud Foundry CLI.

## Versioning

The tile generator uses semver versioning. By default, `tile build` will
generate the next patch release. Major and minor releases can be generated
by explicitly specifying `tile build major` or `tile build minor`. Or to
override the version number completely, specify a valid semver version on
the build command, e.g. `tile build 3.4.5`.

No-op content migration rules are generated for every prior release to the
current release, so that Ops Manager will allow tile upgrades from any
version to any newer version.

## Example

```
$ tile build
name: tibco-bwce
icon: icon.png
label: TIBCO BusinessWorks Container Edition
description: BusinessWorks edition that supports deploying to Cloud Foundry
version: 0.0.2

bosh init release
bosh generate package cf_cli
bosh generate package bwce_buildpack
bosh generate job install_bwce_buildpack
bosh generate job remove_bwce_buildpack
bosh create release --final --with-tarball --version 0.0.2

tile generate release
tile generate metadata
tile generate errand install_bwce_buildpack
tile generate errand remove_bwce_buildpack
tile generate content-migrations

created tile tibco-bwce-0.0.2.pivotal
```

This tile includes a single large buildpack, and takes less than 15 seconds
to build including the CF CLI download and the BOSH release generation.

## Supported Commands

```
init [<tile-name>]
status
build [patch|minor|major|<version>]
clean
clobber
set-name <name>
set-icon <image-file>
set-label <label>
set-description <description>
update-stemcell [<os>] <version>
add-buildpack <buildpack-name> <buildpack-zipfile> [<default-rank>]
update-buildpack <buildpack-name> <buildpack-zipfile> [<default-rank>]
delete-buildpack <buildpack-name>
help
```
