# No Fault Divorce Options

We have identified three different approaches to implementing the functionality required for the no fault divorce legislation.

Services in blue are unchanged, services in green have been re-written, services in yellow have been forked, services in red been modified.

A forked service has been split in a way where there will be two versions maintained separately. A modified service has been changed to support both types of divorce.

## Minimal re-use

This approach would make use of some of the CFT Common Components but none of the existing divorce services.

![mimimal overview](/image/minimal-re-use-overview.mmd.png)

IDAM, Reference Data, Fees and Pay, Send Letter Queue and Docmosis would be re-used as they provide meaningful functionality that would be difficult to replicate in the given timescales and they do not negatively impact the design of the divorce system.

The Document Management Store would not be re-used as it's a thin wrapper around Azure blob and Microsoft provide a fully featured Azure Blob SDK. The infrastructure pipelines contain all the necessary functionality to provide a blob store as part of the pipeline.

CCD would not be used in order to achieve a cleaner architecture, better performance and more autonomy. Instead of using CCD, a new case management service would be created using an open source model where libraries would be used to share functionality.

The Draft Store would not be used as it has been deprecated and it's functionality could be added to the new case management service.

### Advantages

Using a [Domain Driven](https://en.wikipedia.org/wiki/Domain-driven_design) approach a single API could provide the functionality required to manage cases, as opposed to being split between two tightly coupled service: CCD and the Case Orchestrator. This avoids the current requirement to coordinate releases of APIs and CCD definition changes across multiple teams.

CCD does not currently meet the CFT non-functional requirements and is frequently the cause of performance issues. Implementing the case service as a single API would reduce the number of HTTP calls, potentially improve performance and move away from a complicated callback model.

The CCD team currently have a backlog of over 1500 issues and getting them to implement features involves complicated coordination that is not possible in the timescales required for the project.

There is a code freeze that will come into affect from December 2020 to April 2021 meaning any common components will be unable to support new features or changes required for this project. By minimizing the number of common components in use it would be possible to mitigate the impact of this code freeze.

By not tightly coupling to CCD it would also be possible to avoid using ExUI which has its own lengthy backlog. This would allow the team to build a UI that was specific to no fault divorce without having to wait for another team to implement changes.

A simpler architecture would result in a lower maintenance cost and having a custom case service would allow the team to implement new features for the Divorce service faster.

### Disadvantages

While the Central Software Engineering team have successfully prototyped this approach, there remain a number of unknowns making it a high risk option. No fault divorce would be the first service to take this approach and it would have to pay some of the cost to generate standard libraries that other services could then re-use.

There would be a high degree of resistance from certain groups within HMCTS and at this point CCD is very much a known quantity.

It has been said that ExUI is now mandatory for professionals and ExUI is tightly coupled to CCD.

## Some re-use

In order to leverage as much existing code as possible, this approach re-uses all common components, forks the Case Orchestrator, Case Maintenance Service and Document Generation Service. There would be a new petitioner and respondent frontend. The Case Formatter and Evidence Management client API would no longer be used.  

A new CCD case type would created based on the existing one.

![some re-use overview](/image/some-re-use-overview.mmd.png)

The majority of the current case type still applies to no fault divorces so the Case Orchestrator and Case Maintenance Service that handle the relationship with CCD can be re-used with relatively minor modifications.

The petitioner and respondent frontends will be re-written as they are based on legacy libraries and will have to change significantly to support joint applications.

The decree nisi and decree absolute frontends only offer a small amount of functionality and can be rolled into the new frontend in order simplify the architecture and reduce the cost of maintenance.

The Case Formatter is already deemed surplus to requirements by the current divorce team and removing it will solve some long standing technical debt. Likewise, the Evidence Management Client API only exists for historical purposes and can be removed to reduce the maintenance cost.  

The Draft Store would not be used as it has been deprecated and it's functionality is available in CCD.

The Fees and Pay service would be re-used as any fee changes would likely apply to both type of divorce.

The new and existing services would run inside a new no-fault divorce namespace in the CFT kubernetes cluster.

### Advantages

This approach achieves a high level of code re-use by forking the existing codebases and case type. Forking, rather than modification avoids any potential conflicts with multiple teams working on the same codebase.

Re-using large parts of the existing code base will de-risk the project, reduce the time taken to deliver the project and avoid re-doing work that has already been done.

This approach will also reduce the maintenance cost of the service by removing two redundant services without impacting the timescales of the project.

Running the forked services in a separate namespace avoids the potential for services to adversely impact each other.

### Disadvantages

This approach does not address the larger architectural issues of the service. There is still a tight coupling between Case Orchestrator, Case Maintenance Service and the CCD definition file. Any changes to any of these components will result in a change in all of them. This results in large changesets that need to be coordinated and released together.

While both types of divorce case are in use, any changes that impact both will need to be applied twice. In most cases this will just require some monitoring the original source code and applying any relevant patches. In some cases patches might need to be adapted to work in the new codebase.

Depending on common components, such as CCD, adds a risk to the project as any changes they needed to make would likely block the project. Most common components have a lengthy backlog and there is a code freeze that would prevent them from delivering additional features. This risk can be mitigated by avoiding changes to common components but that might impact the quality of the final solution.

The current CCD definition file and divorce services will be actively developed until April so there may be a time where there is quite a lot of synchronisation that needs to occur.

## Maximum re-use

This approach also leverages as much existing code as possible and ties the infrastructure together with the existing divorce service so that any changes that impact both types of divorce can be implemented in a single location.

![maximum re-use overview](/image/maximum-re-use-overview.mmd.png)

A new CCD case type would still be required in order to support the difference in the way the cases progress through the various states. The new case type would be based on the existing case type.

The existing services would be modified in order to support both case types.

The petition frontend would be modified to reflect the new user journey and joint application pathway. The old user journey would be removed from the codebase. The release of this would need to be managed through a long running branch.

In order to avoid changes to the existing divorce service the Draft Store would continue to be used even though it is being phased out.

### Advantages

There would be a slight saving in hosting costs.

This approach would potentially reduce the amount of work required to any changes that affected both types of divorce.

### Disadvantages

While there are some cases where changes can be made across both types of divorce there would still be many cases that led to making the same change in multiple parts of the codebase.

Supporting two case types in a single codebase would significantly increase the complexity. There would need to be branches in the code that detect the case type and adjust the logic accordingly, this is potentially quite fragile and prone to error.

This approach has not been tried before and might have unexpected challenges.

There would be a lot of overhead to coordinate between the existing three teams working on the codebases. There are currently 100 PR on the Orchestrator a month and coordination is already an issue for the team.

The user journey is likely to change so much because of joint applications and the removal of facts it would be difficult to extract the existing user journey through the petitioner frontend in a way that doesn't lead to a re-write.

Maintaining a long running branch is generally regarded as bad practice as it breaks the CI/CD model in place on the programme.

There is a high risk of changes made for one case type inadvertently affecting another. If the new case type is implemented by a different team it would be difficult for them to anticipate all the ways that a change might impact the existing case type.
