---
title: 'Next Release Preview'
linkTitle: 'Next Release Preview'
weight: 2
description:
---

Please make a pull request to describe any changes you wish to highlight
in the next release of Spinnaker. These notes will be prepended to the release
changelog.

## Coming Soon in Release 1.37

### Orca
With https://github.com/spinnaker/orca/pull/4804, the api `/applications/{application}/pipelines?expand=false&limit=2` performance is improved when using sql as the backend.
Orca's queries to its sql db are optimized while handling the data obtained from front50.
TaskController is refactored to support externalized config properties.

https://github.com/spinnaker/orca/pull/4825 adds a new configuration property to orca: `echo.events.ignoreTaskEvents` that defaults to false.  When true, orca doesn't send events whose type is "task".  See <https://spinnaker.io/docs/setup/other_config/features/notifications/#add-a-listening-webhook-to-spinnaker> and <https://spinnaker.io/docs/setup/other_config/features/notifications/#detailstype> for details.


https://github.com/spinnaker/orca/pull/4819 adds the following:
1. **Exclusion of Execution Retrieval for Disabled Pipelines in an Application:**
    - A new configuration has been added to exclude execution retrieval for disabled pipelines in Front50. This can be enabled with:
      ```yaml
      tasks:
         controller:
            excludeExecutionsOfDisabledPipelines: false|true  # Defaults to false
      ```
    - When enabled, Orca will call Front50 with the `enabledPipelines=true` query parameter, which returns only the 
      enabled pipelines for an application (Front50 change https://github.com/spinnaker/front50/pull/1520). This helps reduce
      load for applications with numerous pipelines, especially when obsolete, disabled pipelines are retained for historical reasons.

2. **Agent for Disabling Unused Pipelines:**
    - An agent has been introduced to detect and disable unused or unexecuted pipelines within an application.
      This agent checks pipelines that have not been executed for the past `thresholdDays` days and disables them in Front50.
    - This feature is only available for SQL execution repositories and is configurable with the following settings:
      ```yaml
      pollers:
        unused-pipelines-disable:
          enabled: false | true  # default: false
          intervalSec: 3600  # default: 3600
          thresholdDays: 365  # default: 365
          dryRun: false | true  # default: true. When true an info is logged about the intention to disable a pipelineConfigId in the application evaluated
      ```

### Igor
With https://github.com/spinnaker/igor/pull/1301, GCB CI supports private pool within the same project, addressing the issue in https://github.com/spinnaker/spinnaker/issues/6600 .
With CloudBuild Options in GCB Manifest
```yaml
steps:
  - args:
      - '-c'
      - 'echo hello'
    entrypoint: bash
    name: gcr.io/cloud-builders/git
options:
  pool:
    name: projects/PROJECT_NAME/locations/REGION/workerPools/WORKER_POOL_NAME
```
BuildClient uses location information from the `pool.name` parameter to find the correct worker pool which resulted in 404 earlier.
Depending on the permission setup, it may be able to use build pools outside the project but it does not support other operation, So it is advised not to use build pools outside the project.

#### Breaking changes:
These configuration properties:
```yaml
tasks:
   days-of-execution-history:
   number-of-old-pipeline-executions-to-include:
```
are replaced by the below configuration along with few more newly added ones:
```yaml
tasks:
   controller:
      days-of-execution-history:
      number-of-old-pipeline-executions-to-include:

      optimize-execution-retrieval: <boolean>
      max-execution-retrieval-threads:
      max-number-of-pipeline-executions-to-process:
      execution-retrieval-timeout-seconds:
```
