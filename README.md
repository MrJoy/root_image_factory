# Root Image Factory

This provisions a CI/CD account in a multiaccount AWS setup and then uses that account to create root Debian 11 images.

#### A note on terminology

We use `Root Image` to mean a completely unprovisioned bare image with nothing beyond a basic OS install. We use `Base Image` to mean a partially provisioned image with services required by all operational servers, e.g. monitoring, log aggregation, telemetry, etc.

## Local System Requirements
- terraform >= 1.0
- packer >= 1.7.3
- ansible >= 4.3

## Setup

### Basic Tooling

First setup Homebrew, and make sure everything is up to date.

Clone this repo and `cd` into it.  Run the following to set up tools:

```bash
brew bundle
```

When setting things up, you will need to download a copy of the fork of Packer we use from here: <https://github.com/AlexSc/packer/releases/tag/v1.7.5-dev3>

Unzip and put the file in `~/bin`.

### Secrets

Copy `secrets.auto.pkrvars.hcl.template` to `secrets.auto.pkrvars.hcl`, and populate all the variables listed within.

## Provisioning

We use terraform workspaces, and the terraform module is configured such that use of the default workspace is invalid. The development workspace will configure and use our development Infrastructure AWS account, and the production workspace will correspondingly use our production Infrastructure AWS account.

Select a workspace with `aws-vault exec mrjoy -- terraform workspace select <env>`

Then run `aws-vault exec mrjoy -- terraform plan -var-file=<workspace>.tfvars -out=run.tfplan`. Verify the proposed modifications. If they all look good, run `terraform apply run.tfplan` to apply changes.

After running terraform, change to the packer directory and run

```bash
aws-vault exec mrjoy -- ~/bin/packer_1.7.5-dev3_darwin_arm64 build --var-file=root_image.auto.pkrvars.hcl --var-file=../secrets.auto.pkrvars.hcl -var build_account_canonical_slug=stage-ci-cd -timestamp-ui -except "vagrant.*" root_image.pkr.hcl
```

to build root Debian 11 AMIs and Vagrant boxes.
