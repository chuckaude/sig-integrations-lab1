# SIG Integration Lab 1
The goal of this lab is provide hands on experience configuring a Polaris workflow in GitHub and viewing the results. As part of the lab, we will execute a full scan which will have a policy induced failure. We will look at a Fix PR for a vulnerable component (log4j). And we will review the code scanning findings in the GitHub Advanced Security tab. 

This repo contains everything you need to do the lab with the exception of the two prerequisites listed below.

# Prerequisites

1. GitHub Account - signup at https://github.com/signup
2. Polaris Account and Access Token - https://polaris.synopsys.com/developer/default/polaris-documentation/t_make-token

# Clone repository

1. Clone this repository into your GitHub account (_GitHub → New → Import a Repository_) **Milestone 1** :heavy_check_mark:
   - enter https://github.com/chuckaude/sig-integrations-lab1.git
   - enter repo name, e.g. hello-java
   - leave as public (required for GHAS)

# Setup workflow

1. Confirm GITHUB_TOKEN has workflow read & write permissions (_GitHub → Project → Settings → Actions → General → Workflow Permissions_)
2. Confirm all GitHub Actions are allowed (_GitHub → Project → Settings → Actions → General → Actions Permissions_)
3. Add the following variables, adding POLARIS_ACCESSTOKEN as a **secret** (_GitHub → Project → Settings → Secrets and Variables → Actions_)
- POLARIS_SERVERURL
- POLARIS_ACCESSTOKEN

4. Add a coverity.yaml to the project repository (_GitHub → Project → Add file → Create new file_)

```
capture:
  build:
    clean-command: mvn -B clean
    build-command: mvn -B -DskipTests package
analyze:
  checkers:
    webapp-security:
      enabled: true
```

5. Create an application via the Polaris UI and assign SAST and SCA subscriptions. Note: application name must match what is defined in the workflow, e.g. chuckaude-hello-java (replace prefix with your name)
6. Create a new workflow (_GitHub → Project → Actions → New Workflow → Setup a workflow yourself_) **Milestone 2** :heavy_check_mark:

```
# example workflow for Polaris scans using the Synopsys Action
# https://github.com/marketplace/actions/synopsys-action
name: polaris
on:
  push:
    branches: [ main, master, develop, stage, release ]
  pull_request:
    branches: [ main, master, develop, stage, release ]
  workflow_dispatch:
jobs:
  polaris:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout Source
      uses: actions/checkout@v4
    - name: Setup Java JDK
      uses: actions/setup-java@v4
      with:
        java-version: 17
        distribution: microsoft
        cache: maven
    - name: Polaris Scan
      uses: synopsys-sig/synopsys-action@v1.9.0
      with:
        polaris_server_url: ${{ vars.POLARIS_SERVERURL }}
        polaris_access_token: ${{ secrets.POLARIS_ACCESSTOKEN }}
        polaris_assessment_types: 'SAST,SCA'
        polaris_application_name: chuckaude-${{ github.event.repository.name }}
        polaris_project_name: ${{ github.event.repository.name }}
        polaris_prComment_enabled: 'true'
        polaris_reports_sarif_create: 'true'
        polaris_upload_sarif_report: 'true'
        github_token: ${{ secrets.GITHUB_TOKEN }}
#    - name: Save Logs
#      if: always()
#      uses: actions/upload-artifact@v4
#      with:
#        name: bridge-logs
#        path: ${{ github.workspace }}/.bridge
```
# Full Scan

1. Monitor your workflow run and wait for scan to complete (_GitHub → Project → Actions → Polaris → Most recent workflow run → Polaris_)
2. Once workflow completes, select _Summary_ in upper left to see policy enforment / break the build / workflow failure **Milestone 3** :heavy_check_mark:
3. View the Fix PR for vulnerable log4j dependency (_GitHub → Project → Pull requests → Synopsys Automated PR_) **Milestone 4** :heavy_check_mark:
4. View finding in GitHub Advanced Security tab (_GitHub → Project → Security → Code scanning_) **Milestone 5** :heavy_check_mark:

# PR scan

1. Edit pom.xml (_GitHub → Project → Code → pom.xml → Edit pencil icon upper right_)
   - change log4j version from 2.14.1 to 2.15.0
3. Click on _Commmit Changes_, select create a **new branch** and start a PR
4. Review changes and click on _Create Pull Request_
5. Monitor workflow run (_GitHub → Project → Actions → Polaris → Most recent workflow run → Polaris_)
6. Once workflow completes, navigate back to PR and see PR comment **Milestone 6** :heavy_check_mark: (_GitHub → Project → Pull requests)

# Congratulations

You have now configured a Polaris workflow in GitHub and demonstrated all the current post-scan CI features. :clap: :trophy:
