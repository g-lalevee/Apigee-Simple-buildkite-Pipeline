[![Build status](https://badge.buildkite.com/a2300b51ad050dd890aa263ef0fbe3fa30103ebb2f80d3b1e3.svg)](https://buildkite.com/apigee/apigee-simple-buildkite-pipeline)
[![PyPI status](https://img.shields.io/pypi/status/ansicolortags.svg)](https://pypi.python.org/pypi/ansicolortags/) 

# # Apigee-Simple-buildkite-Pipeline

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
  - The authentication to the Apigee X / Apigee hybrid management API is done using a GCP Service Account. See CI/CD Configuration [Instructions](https://github.com/clalevee/apigee-simple-buildkite-pipeline-v2#CI/CD-Configuration-Instructions).

## CI/CD Configuration Instructions

To be continued... Work in progress...