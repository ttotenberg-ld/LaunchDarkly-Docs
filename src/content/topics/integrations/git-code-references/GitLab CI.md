---
title: "GitLab CI"
excerpt: ""
---
## Overview
This topic explains how to set up and configure the Gitlab CI to use with LaunchDarkly.

You can use the [`ld-find-code-refs`](https://github.com/launchdarkly/ld-find-code-refs/) utility with [GitLab CI](https://about.gitlab.com/product/continuous-integration/) to automatically populate code references in LaunchDarkly. 

Follow the procedure below to create a GitLab CI configuration using LaunchDarkly's code references executable.
## Prerequisites
To complete this procedure, you must have the following prerequisites:

* A personal API access token with `writer` permissions. To learn more, read [Personal API access tokens](./api-access-tokens).
## Setting up GitLab CI
Here's how to set up the GitLab CI:


1. Navigate to your GitLab project's CI / CD settings by clicking through **Your project** > **Settings** > **CI/CD**. 
2. Expand **Variables**. 
3. Create a variable called `LD_ACCESS_TOKEN`. Use the same value as your LaunchDarkly access token. Click the toggle to set the variable to **Masked**.
4. Create a variable called `LD_PROJECT_KEY`. Use your LaunchDarkly project's key as the value. To learn more about setting variables, read [GitLab's documentation](https://docs.gitlab.com/ee/ci/variables/#creating-a-custom-environment-variable).
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/d4eaec5-_Code_Refs_Example__GitLab_2019-10-03_09-55-33.png",
        " Code Refs Example · GitLab 2019-10-03 09-55-33.png",
        1082,
        446,
        "#f2f3f3"
      ],
      "caption": "The GitLab Variables screen with masked `LD_ACCESS_TOKEN` and `LD_PROJECT_KEY` created."
    }
  ]
}
[/block]
5. Open your `.gitlab-ci.yml` file. This file defines your project's CI/CD pipeline. To learn more about getting started with GitLab CI, read [GitLab's documentation](https://docs.gitlab.com/ee/ci/#getting-started).
6. Copy and paste the following into `.gitlab-ci.yml`. No changes to the script are needed if your pipeline runs on Alpine Linux. If `apk` is unavailable in your environment then you'll need to modify the first three steps to use a different package manager.
[block:code]
{
  "codes": [
    {
      "code": "find-launchdarkly-code-refs:\n  stage: deploy\n  script:\n    # Update package manager\n    - apk update\n    # Install packages needed to download the Find Code Refs tool\n    - apk add curl grep wget \n    # Install packages used by the Find Code Refs tool\n    - apk add the_silver_searcher git\n    # Download the Find Code Refs tool\n    - curl -s https://api.github.com/repos/launchdarkly/ld-find-code-refs/releases/latest\n       | grep browser_download_url | grep linux | grep amd64\n       | cut -d '\"' -f 4\n       | wget -qi -\n    # Unpackage the Find Code Refs tool\n    - tar xvzf ld-find-code-refs*.tar.gz\n    # Find LaunchDarkly code references in your source code\n    - ./ld-find-code-refs\n       -accessToken $LD_ACCESS_TOKEN\n       -projKey $LD_PROJECT_KEY\n       -dir $CI_PROJECT_DIR\n       -repoName $CI_PROJECT_NAME\n       -repoUrl https://gitlab.com/$CI_PROJECT_PATH\n       -branch $CI_COMMIT_REF_NAME\n       -updateSequenceId $CI_PIPELINE_IID\n       -commitUrlTemplate https://gitlab.com/${CI_PROJECT_PATH}/commit/\"\\${sha}\"\n       -hunkUrlTemplate https://gitlab.com/${CI_PROJECT_PATH}/blob/\"\\${sha}/\\${filePath}#L\\${lineNumber}\"\n",
      "language": "yaml"
    }
  ]
}
[/block]

## How the script works
When executed, this script downloads and runs the `ld-find-code-refs` utility. 

This script:
* Installs the necessary dependencies,
* Downloads and unpacks the utility, and
* Runs the utility with the previously-set variables, as well as GitLab-specific configurations.

The `find-launchdarkly-code-refs` script runs in GitLab's `deploy` phase. As written, `find-launchdarkly-code-refs` runs concurrent to other scripts in the `deploy` stage. We positioned the script this way so problems running `ld-find-code-refs` won't block the deployment pipeline. 

In the example `.gitlab-ci.yml` below, the `find-launchdarkly-code-refs` script runs as a part of a project's pipeline.
[block:code]
{
  "codes": [
    {
      "code": "image: alpine:latest
build1:\n  stage: build\n  script:\n    - echo \"Build something\"
test1:\n  stage: test\n  script:\n    - echo \"Test something\"\n    \ndeploy1:\n  stage: deploy\n  script:\n    - echo \"Deploy something\"\n    \nfind-launchdarkly-code-refs:\n  stage: deploy\n  script:\n    # Update package manager\n    - apk update\n    # Install packages needed to download the Find Code Refs tool\n    - apk add curl grep wget \n    # Install packages used by the Find Code Refs tool\n    - apk add the_silver_searcher git\n    # Download the Find Code Refs tool\n    - curl -s https://api.github.com/repos/launchdarkly/ld-find-code-refs/releases/latest\n       | grep browser_download_url | grep linux | grep amd64\n       | cut -d '\"' -f 4\n       | wget -qi -\n    # Unpackage the Find Code Refs tool\n    - tar xvzf ld-find-code-refs*.tar.gz\n    # Find LaunchDarkly code references in your source code\n    - ./ld-find-code-refs\n       -accessToken $LD_ACCESS_TOKEN\n       -projKey $LD_PROJECT_KEY\n       -dir $CI_PROJECT_DIR\n       -repoName $CI_PROJECT_NAME\n       -repoUrl https://gitlab.com/$CI_PROJECT_PATH\n       -branch $CI_COMMIT_REF_NAME\n       -updateSequenceId $CI_PIPELINE_IID\n       -commitUrlTemplate https://gitlab.com/${CI_PROJECT_PATH}/commit/\"\\${sha}\"\n       -hunkUrlTemplate https://gitlab.com/${CI_PROJECT_PATH}/blob/\"\\${sha}/\\${filePath}#L\\${lineNumber}\"\n",
      "language": "yaml"
    }
  ]
}
[/block]
When the jobs run in the pipeline, they display like this:
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/b0af00f-_Code_Refs_Example__GitLab_2019-10-03_10-54-20.png",
        " Code Refs Example · GitLab 2019-10-03 10-54-20.png",
        1442,
        358,
        "#f7f8f7"
      ],
      "caption": "A screenshot of the jobs running."
    }
  ]
}
[/block]

## Additional configuration options
There are more configuration options for `ld-find-code-refs`. 

To learn more, read [Optional arguments](https://github.com/launchdarkly/ld-find-code-refs#optional-arguments).