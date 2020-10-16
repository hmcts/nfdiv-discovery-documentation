# Overview of Divorce Service

The current divorce service was one of the first delivered as part of CFT Reform. It has been developed and maintained by multiple suppliers during it's lifetime and currently has three teams actively working on it.

## System overview

The system is comprised of six APIs written in Java and four frontends written in node.js. It makes use of several common components provided by the platform.

![divorce overview](/image/as-is-overview.mmd.png)

### Frontends

The frontend services are all node.js based frontends of various ages. The petitioner frontend is quite out of date and relies on several legacy libraries.

### Case Orchestrator Service ([COS](https://github.com/hmcts/div-case-orchestration-service))

The Case Orchestrator is the primary service within Divorce. It is currently maintained by three separate teams and receives over 100 pull requests a month.

It is primarily responsible for managing the state transitions of cases in CCD. It handles all CCD callbacks.

The Case Orchestrator has a number of Quartz Scheduler jobs that are stored in a postgres database.

It uses up-to-date libraries and appears well tested and maintained.

### Case Maintenance Service ([CMS](https://github.com/hmcts/div-case-maintenance-service))

The Case Maintenance Service was added to abstract communication between the Case Orchestrator and CCD or Draft Store.

It was added after the Case Orchestrator was created and so further investigation needs to be made to verify how often it is used.

Given the tightly coupled nature of CCD and the Case Orchestrator it does not seem a worthwhile venture trying to add a layer of abstraction between them. Especially as the Case Orchestrator handles the callbacks from CCD.

### Case Formatter Service ([CFS](https://github.com/hmcts/div-case-data-formatter))

The Case Formatter does some minimal cleaning of case data. The Tech Lead has tried to remove this service but was unable to do so before being moved on to delivering features.

It is understood that the functionality this service delivers is trivial, such as removing duplicate documents from a list of documents.

The Tech Lead believes it should be collapsed into the Case Orchestrator Service.

### Document Generation Service ([DGS](https://github.com/hmcts/div-document-generator-client))

The Document Generation Service is a proxy in front of the Docmosis common component.

It may also house some legacy HTML templates.

The Tech Lead feels that this service does provide some value by aiding in debugging and regenerating documents.

### Evidence Management Client API ([EMC](https://github.com/hmcts/div-evidence-management-client-api))

The Evidence Management Client API is a proxy for the Document Management Store API (Doc Store) provided by the Evidence Management team.

It only implements a single API call and looks redundant. The most likely explanation for is existence is that some of the Divorce services may predate the Doc Store.

The Tech Lead does not believe it required.

### Fees and Pay

Proxy. Orchestrator connects direct to Pay but uses this proxy for Fees.

## Approach to development

There is no local development environment for divorce. It was decided that the requirements of running CCD locally were too resource intensive so development is primarily done with local unit testing or connecting to AAT to use services there.

The divorce team are looking to reduce the number of integration tests as unreliable common components have frequently broken the build and blocked releases. Instead the team intend to rely on unit tests.

## Infrastructure and pipelines

The divorce service has uses the standard CFT Reform approach to infrastructure. The [shared-infrastructure](https://github.com/hmcts/div-shared-infrastructure) deploys a shared vault and appinsights instance. There is also an [action group mailing list](https://github.com/hmcts/div-shared-infrastructure/blob/master/action-groups.tf).

There is a [helm product chart](https://github.com/hmcts/chart-div) but it is not used by any of the divorce services.

The CCD definition file is maintained in it's [own codebase](https://github.com/hmcts/div-ccd-definitions) that has a build pipeline that automates deployment of any changes to the definition file to AAT.

There are some automated regression tests that log in to CCD and upload the definition file. It's not clear whether the intention is to expand that and add tests that start progressing cases.

The definition file repository makes use of the CCD product chart to deploy CCD to preview in order to test changes in a pull request.
