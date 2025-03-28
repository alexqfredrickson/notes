<!-- from Continuous Delivery by Jez Humble / David Farley -->


# Chapter 1: Foundations

Deployment pipelines involve building binaries, testing those binaries, deploying those binaries, and releasing binaries.

Manual deployments, deploying straight from development to production, and manual configuration management are all considered anti-patterns.

Every code change should trigger a CI pipeline - building and testing each artifact.

Testing should include unit testing, ensuring test coverage, functional acceptance testing, non-functional testing (i.e. capacity, availability, and security testing), and demoing to end-users. 

Complex distributed system deployments, including data migration, should take around 5-20 minutes. In some organizations, that can take around 30 days.


# Chapter 2: Configuration Management

Configuration management implies that all environments, including OS versions/patches, network configurations, software, applications, and configuration deployed to them, can be reproduced exactly. It also implies that incremental changes can be made to each of these, and that those changes are auditable, so they can satisfy regulatory compliance.

Build scripts can pull configuration at build-time, packaging software can inject configuration at packaging time, deployment scirpts can pass configuration at deploy time, and the application can fetch configuration at runtime. The authors believe that configuration supply should be consistent across a single one of these mediums.

The authors also state that:

  * Everything should be in version control (including even *release plans*, the authors say - thus mildly overstating the case) - except for binary build output.
  * All changes should be checked into `main` as often as possible, and advocate against the use of branches on the basis that they (a) defer integration of new functionality and (b) only uncover integration issues on-main-merge.
  * All dependencies should be pinned, as unpinned dependencies mean non-reproducible builds.
  * Monolithic applications should be componentized.
  * Configuration is equally risky to change compared to source code. 
  * Too much configurability is a bad thing and it kills projects.


# Chapter 3: Continuous Integration

<!-- todo: pg. 55 -->