# metal-runner-action

[![Experimental](https://img.shields.io/badge/Stability-Experimental-red.svg)](https://github.com/equinix-labs/equinix-labs/blob/main/experimental-statement.md#experimental-statement)

[![GitHub Super-Linter](https://github.com/equinix-labs/metal-action-runner/actions/workflows/superlinter.yml/badge.svg)](https://github.com/marketplace/actions/super-linter)

This Experimental GitHub Action sets up an Equinix Metal server that can be used to run code in your workflows.

> :bulb: See also:
>
> - [equinix-metal-device](https://github.com/equinix-labs/metal-device-action) action
> - [equinix-metal-project](https://github.com/equinix-labs/metal-project-action) action
> - [equinix-metal-sweeper](https://github.com/equinix-labs/metal-sweeper-action) action
> - WIP [equinix-metal-github-actions-examples](https://github.com/equinix-labs/metal-actions-example) examples

## Inputs

| Name                   | Description                             | Required | Default |
| ---------------------- | --------------------------------------- | -------- | ------- |
| `github_token`         | GitHub token with repository scope      | Yes      | -       |
| `metal_auth_token`     | API Key for Equinix Metal               | Yes      | -       |
| `metal_project_id`     | Project ID for Equinix Metal            | Yes      | -       |
| `metro`                | Metro for Equinix Metal                 | Yes      | -       |
| `plan`                 | Plan for Equinix Metal                  | Yes      | -       |
| `os`                   | OS for Equinix Metal                    | Yes      | -       |
| `provisioning_timeout` | How long to wait for provisioning (min) | No       | 30      |

## Usage

```yaml
name: CI Pipeline

on:
  push:
    branches:
      - main

jobs:
  Project:
    name: "Create Project"
    runs-on: ubuntu-latest

    steps:
      - name: metal-project-action
        id: metal-project-action
        uses: equinix-labs/metal-project-action@v0.12.1
        with:
          projectName: "metal-runner-demo"
          userToken: ${{ secrets.METAL_AUTH_TOKEN }}

    outputs:
      projectID: ${{ steps.metal-project-action.outputs.projectID }}

  Runner:
    name: "Create Runner"
    needs: Project
    runs-on: ubuntu-latest

    steps:
      - name: metal-runner-action
        uses: equinix-labs/meta-action-runner@v0.1.0
        with:
          github_token: ${{ secrets.TEST_PAT_KEY }}
          metal_auth_token: ${{ secrets.METAL_AUTH_TOKEN }}
          metal_project_id: ${{ needs.Project.outputs.projectID }}
          metro: "da"
          plan: "c3.small.x86"
          os: "ubuntu_20_04"

  Demo:
    name: "Demo Action"
    needs: Runner
    runs-on: self-hosted

    steps:
      - run: |
          echo "Hello, Equinix Metal!"
          echo "This is runner: ${{ runner.name }}"
          echo "Running on ${{ runner.arch }} ${{ runner.os }}"

  Cleanup:
    name: "Cleanup"
    runs-on: ubuntu-latest
    needs: [Demo, Project]

    steps:
      - name: metal-sweeper-action
        uses: equinix-labs/metal-sweeper-action@v0.5.1
        with:
          authToken: ${{ secrets.METAL_AUTH_TOKEN }}
          projectID: ${{ needs.Project.outputs.projectID  }}
          keepProject: false
```

## Support

This repository is [Experimental](https://github.com/equinix-labs/equinix-labs/blob/main/experimental-statement.md) meaning that it's based on untested ideas or techniques and not yet established or finalized or involves a radically new and innovative style! This means that support is best effort (at best!) and we strongly encourage you to NOT use this in production.
