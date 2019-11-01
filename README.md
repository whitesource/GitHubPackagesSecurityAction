# Whitesource GPR Security Action
This action is designed to run as part of the workflow "registry_package" [triggered event](https://help.github.com/en/github/automating-your-workflow-with-github-actions/events-that-trigger-workflows).

It scans the published/updated Docker image in GPR and reports back with found security vulnerabilities and license information.

# Usage
See [action.yml](action.yml)

### Input Parameters
**gpr-token**: GitHub personal access token with read/write privileges to GPR. This parameter must be a [repository secret](https://help.github.com/en/github/automating-your-workflow-with-github-actions/virtual-environments-for-github-actions#creating-and-using-secrets-encrypted-variables). Required parameter.

**ws-destination-url**: WhiteSource environment destination url. Required parameter.

**ws-api-key**: WhiteSource organization api key. This parameter must be a [repository secret](https://help.github.com/en/github/automating-your-workflow-with-github-actions/virtual-environments-for-github-actions#creating-and-using-secrets-encrypted-variables). Required parameter.

**ws-user-key**: WhiteSource user key. This parameter must be a [repository secret](https://help.github.com/en/github/automating-your-workflow-with-github-actions/virtual-environments-for-github-actions#creating-and-using-secrets-encrypted-variables). Required parameter.

**ws-product-key**: WhiteSource product key to publish results to. This parameter must be a [repository secret](https://help.github.com/en/github/automating-your-workflow-with-github-actions/virtual-environments-for-github-actions#creating-and-using-secrets-encrypted-variables). If not specified - a default product will be created.

**print-scan-report**: Whether to print the results report as part opf the action's log. Default is false.

**actions_step_debug**: Whether to print debug logs. Default is false.

### Output Parameters
**scan-report-file-path**: Path of the scan report file.

**scan-report-folder-path**: Path of the folder of the scan report file.

### Workflow Examples
The recommended way to add this action to your workflow, is with a subsequent action that uploads the report json as an artifact. For example:
```yaml
on: registry_package
name: WORKFLOW_NAME
jobs:
  gprSecurityJob:
    name: GPR Security Check Job
    runs-on: ubuntu-latest
    steps:
      - name: GPR Security Check Step
        id: gpr-security-check
        uses: whitesource/GprSecurityAction@19.10.2
        with:
          gpr-token: ${{ secrets.GPR_ACCESS_TOKEN }}
          ws-api-key: ${{ secrets.WS_API_KEY }}
          ws-user-key: ${{ secrets.WS_USER_KEY }}
          ws-product-key: ${{ secrets.WS_PRODUCT_KEY }}
          ws-destination-url: https://saas.whitesourcesoftware.com/agent
      - name: Upload Report
        uses: actions/upload-artifact@master
        with:
          name: security-scan-log
          path: ${{ steps.gpr-security-check.outputs.scan-report-folder-path }}
```

Another option is to print the scan report to the step's log, without uploading it as an artifact:
```yaml
on: registry_package
name: WORKFLOW_NAME
jobs:
  gprSecurityJob:
    name: GPR Security Check Job
    runs-on: ubuntu-latest
    steps:
      - name: GPR Security Check Step
        id: gpr-security-check
        uses: whitesource/GprSecurityAction@19.10.2
        with:
          gpr-token: ${{ secrets.GPR_ACCESS_TOKEN }}
          ws-api-key: ${{ secrets.WS_API_KEY }}
          ws-user-key: ${{ secrets.WS_USER_KEY }}
          ws-product-key: ${{ secrets.WS_PRODUCT_KEY }}
          ws-destination-url: https://saas.whitesourcesoftware.com/agent
          print-scan-report: true
```


# Scan Report File
The output is a report in json format, which includes information on vulnerabilities, license, top fixes and inventory details. For example:
```json
{
  "projectVitals": {
    "productName": "demo product",
    "name": "demo project",
    "token": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  },
  "libraries": [
    {
      "keyUuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "keyId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "type": "REDHAT_PACKAGE_MODULE",
      "languages": "RPM",
      "references": {
        "url": "http://mirror.centos.org/centos/7/os/x86_64/Packages/sqlite-3.7.17-8.el7.x86_64.rpm",
        "homePage": "http://www.sqlite.org/"
      },
      "outdated": true,
      "sha1": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "name": "sqlite.rpm",
      "artifactId": "sqlite.rpm",
      "version": "3.7.17-8.el7",
      "groupId": "sqlite",
      "licenses": [
        {
          "name": "Public Domain",
          "url": "http://creativecommons.org/licenses/publicdomain/",
          "profileInfo": {
            "copyrightRiskScore": "ONE",
            "patentRiskScore": "THREE",
            "copyleft": "NO",
            "linking": "NON_VIRAL",
            "royaltyFree": "NO"
          },
          "referenceType": "RPM (details available in package spec file)",
          "reference": "packageName\u003dsqlite\u0026url\u003dhttp://mirror.centos.org/centos/7/os/x86_64/Packages/sqlite-3.7.17-8.el7.x86_64.rpm"
        }
      ],
      "vulnerabilities": [
        {
          "name": "CVE-2018-8740",
          "type": "CVE",
          "severity": "MEDIUM",
          "score": 5.0,
          "cvss3_severity": "HIGH",
          "cvss3_score": 7.5,
          "scoreMetadataVector": "CVSS:3.0/AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:N/A:H",
          "publishDate": "2018-03-17",
          "url": "https://cve.mitre.org/cgi-bin/cvename.cgi?name\u003dCVE-2018-8740",
          "description": "In SQLite through 3.22.0, databases whose schema is corrupted using a CREATE TABLE AS statement could cause a NULL pointer dereference, related to build.c and prepare.c.",
          "allFixes": [],
          "references": []
        }
      ]
    }
  ]
}
```

# License

The scripts and documentation in this project are released under the [APACHE 2.0](LICENSE)
