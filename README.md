[![Build status](https://badge.buildkite.com/a2300b51ad050dd890aa263ef0fbe3fa30103ebb2f80d3b1e3.svg)](https://buildkite.com/apigee/apigee-simple-buildkite-pipeline)
[![PyPI status](https://img.shields.io/pypi/status/ansicolortags.svg)](https://pypi.python.org/pypi/ansicolortags/) 

## Apigee-Simple-buildkite-Pipeline

**This is not an official Google product.**<BR>This implementation is not an official Google product, nor is it part of an official Google product. Support is available on a best-effort basis via GitHub.

***
  

## Goal

Simple implementation for a CI/CD pipeline for Apigee using GitHub repository, 
[CI/CD with Buildkite](https://buildkite.com/docs/pipelines) and the Apigee Maven Plugins.

The CICD pipeline includes:

- Git branch dependent Apigee environment selection and proxy naming to allow
  deployment of feature branches as separate proxies in the same environment
- Open API Specification (Swagger) static code analysis using [stoplight spectral](https://github.com/stoplightio/spectral)
- Static Apigee Proxy code analysis using [apigeelint](https://github.com/apigee/apigeelint)
- Static JS code analysis using [eslint](https://eslint.org/)
- Unit JS testing using [mocha](https://mochajs.org/)
- Integration testing of the deployed proxy using
  [apickli](https://github.com/apickli/apickli)
- Packaging and deployment of an Apigee configuration using
  [Apigee Config Maven Plugin](https://github.com/apigee/apigee-config-maven-plugin)
- Packaging and deployment of the API proxy bundle using
  [Apigee Deploy Maven Plugin](https://github.com/apigee/apigee-deploy-maven-plugin)

### API Proxy and Apigee configuration

The folder [./apiproxy](./apiproxy) includes a simple API proxy bundle, a simple Apigee configuration file [./EdgeConfig/edge.json](./EdgeConfig/edge.json) as well as the following resources:

- [Buildkite Pipeline File](.buildkite/pipeline.yml) to define a Buildkite CI
  multi-branch pipeline.
- [specs Folder](./specs) to hold the specification file for provided proxy.
- [test Folder](./test) to hold the specification (owasp ruleset), unit and integration tests.

## Target Audience

- Operations
- API Engineers
- Security

## Limitations & Requirements

  - The authentication to the Apigee Edge management API is done using OAuth2. If you require MFA, please see the [documentation](https://github.com/apigee/apigee-deploy-maven-plugin#oauth-and-two-factor-authentication) for the Maven deploy plugin for how to configure MFA.
  - The authentication to the Apigee X / Apigee hybrid management API is done using a GCP Service Account. See CI/CD Configuration Instructions.

## CI/CD Configuration Instructions

### Create GCP Service Account (Apigee hybrid / Apigee X only)

Apigee hybrid / Apigee X deployement requires a GCP Service Account with the following roles (or a custom role with all required permissions):

- Apigee API Admin
- Apigee Environment Admin

To create it in your Apigee organization's GCP project, use following gcloud commands (or GCP Web UI):

```sh
SA_NAME=<your-new-service-account-name>

gcloud iam service-accounts create $SA_NAME --display-name="Buildkite Service Account"

PROJECT_ID=$(gcloud config get-value project)
BUILDKITE_SA=$SA_NAME@$PROJECT_ID.iam.gserviceaccount.com

gcloud projects add-iam-policy-binding "$PROJECT_ID" \
  --member="serviceAccount:$BUILDKITE_SA" \
  --role="roles/apigee.environmentAdmin"

gcloud projects add-iam-policy-binding "$PROJECT_ID" \
  --member="serviceAccount:$BUILDKITE_SA" \
  --role="roles/apigee.apiAdmin"

gcloud iam service-accounts keys create $SA_NAME-key.json --iam-account=$BUILDKITE_SA --key-file-type=json 

```

Later, you will have to copy the `<your-new-service-account-name>-key.json` file to your Buildkite Agent **hooks** folder.


### Initialize a GitHub Repository

Create a GitHub repository to hold your API Proxy. 

To use the `Apigee-Simple-buildkite-Pipeline`
in your GitHub repository like `github.com/my-user/my-api-proxy-repo`, follow these
steps:

```bash
git clone git@github.com:g-lalevee/Apigee-Simple-buildkite-Pipeline.git
cd Apigee-Simple-buildkite-Pipeline 
git init
git remote add origin git@github.com:my-user/my-api-proxy.git
git checkout -b feature/cicd-pipeline
git add .
git commit -m "initial commit"
git push -u origin feature/cicd-pipeline
```

## Buildkite Pipeline Configuration 

1.  Create Buildkite Pipeline
    - From Buildkite interface, create a new pipeline, and link it to your GitHub repository. See [Builtkit Create Pipeline documentation](https://buildkite.com/docs/pipelines/create-your-own#create-a-pipeline)

    - Configure pipeline to use pipeline configuration file .built.kite/pipeline.yml, from your GitHub repository. See [Builtkit Pipeline Configuration documentation](https://buildkite.com/docs/pipelines/defining-steps#step-defaults-yaml-steps-editor).

2.  Create Buildkite Agent
    - If you don't have an agent, create one. See [Builtkit Agent documentation](https://buildkite.com/organizations/apigee/agents).
  
3.  Configure Buildkite Secret Management

    To be to connect to Apigee, we need to store credentials: Service Account key file for Apigee X/hybrid or UserID and Password for Apigee Edge. See [Builtkit Secret Management documentation](https://buildkite.com/docs/pipelines/secrets).<BR> 
    In this sample, we will store credentials and export secrets with environment hooks:<BR>

    - In Buildkite Agent configuration folder, **hooks** folder, edit **environment** file and add the following lines:

      ```
      #!/bin/bash

      set -euo pipefail

      echo "_____Running environment hook" $BUILDKITE_PIPELINE_SLUG

      # ---------------------------------------------
      # For Apigee X/hybrid, export GCP Service 
      # Account key file
      # ---------------------------------------------

      if [[ "$BUILDKITE_PIPELINE_SLUG" == "apigee-simple-buildkite-pipeline" &&  "$BUILDKITE_LABEL" == "Generate SA key file" ]]; then
        # base64 /opt/homebrew/etc/buildkite-agent/hooks/sa.json > ./sa.txt
        base64 -i /opt/homebrew/etc/buildkite-agent/hooks/sa.json -o ./sa.txt
        buildkite-agent artifact upload ./sa.txt
      fi

      # ---------------------------------------------
      # For Apigee Edge, export Apigee credentials 
      # (UserID and Password)
      # ---------------------------------------------

      if [[ "$BUILDKITE_PIPELINE_SLUG" == "apigee-simple-buildkite-pipeline" && ( "$BUILDKITE_LABEL" == "mvn deploy cfg edge"  || "$BUILDKITE_LABEL" == "mvn package & deploy proxy Edge" ) ]]; then
        export APIGEE_CREDS_USR='YOUR_APIGEE_EDGE_USER_ID'
        export APIGEE_CREDS_PASSWORD='YOUR_APIGEE_EDGE_PASSWORD'
      fi
      ```
      > Note: this script was created and tested on MacOS.

    - For Apigee X/hybrid, copy your Service Account key file to **hooks** folder.
    - For Apigee Edge, replace Apigee UserName and Password values by your own, in the script.

    > Note: To authenticate to Apigee X:hybrid, you can also use GCP [Workload Identity Federation](https://cloud.google.com/iam/docs/workload-identity-federation).
    > Buildkite providesplugins to integrate reading and exposing secrets to your build steps using secrets storage services, including Workload Identity Federation. See [Buildkite  Workload Identity Federation repository](https://github.com/buildkite-plugins/gcp-workload-identity-federation-buildkite-plugin).

  ## Run the pipeline

Using your favorite IDE...
1.  Update the **.buildkite/pipeline.yml** file<BR>
In global **env:** section, change **DEFAULT_APIGEE_ORG**, **DEFAULT_APIGEE_ENV**, **TEST_HOST** values by your target Apigee organization, environment and Apigee environment hostname.<BR>
Update **API_VERSION** variable to define Apigee target: `googleapi` = Apigee X / Apigee hybrid, `apigeeapi` = Apigee Edge
2.  Read carefully the **Set Deployment Target** step to check if the multibranch rules match your Git and Apigee environment naming and configuration.
3. Save
4. Commit, Push.. et voila!


    