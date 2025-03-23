<!-- from Continuous Delivery by Jez Humble / David Farley -->


# Chapter 1: Foundations

Deployment pipelines involve building binaries, testing those binaries, deploying those binaries, and releasing binaries.

Manual deployments, deploying straight from development to production, and manual configuration management are all considered anti-patterns.

Every code change should trigger a CI pipeline - building and testing each artifact.

Testing should include unit testing, ensuring test coverage, functional acceptance testing, non-functional testing (i.e. capacity, availability, and security testing), and demoing to end-users. 

Complex distributed system deployments, including data migration, should take around 5-20 minutes. In some organizations, that can take around 30 days.


# Chapter 2: Configuration Management

<!-- todo: pg. 31 -->