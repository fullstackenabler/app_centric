# Application-Centric Enablement (ACE)

The "application-centric" enablement methodology provides students with a framework that is built around a specific application and the ecosystem of requirements and foundational technologies that define it. This is represented in the following diagram.

<img src="https://github.com/cmangubat/app_centric/assets/43074193/80d52536-1e2f-41e8-a03f-2ce633c2a1ab.png" width="500" height="243">

An enterprise has a need. To satisfy this need, the enterprise needs one or more applications. If the enterprise sees fit to create this application itself (as would be the case in DevOps oriented enterprises), they would do so using a number of underlying technologies that make the application possible.

A technical audience, that needs to support or sell this application, therefore needs to knowledgeable in the following areas:

<img src="https://github.com/cmangubat/app_centric/assets/43074193/3af7ae0a-1781-412d-95db-ab19def012d5.png" width="500" height="243">

The audience must:

- Understand why the application is needed
- Understand what needs to be protected (e.g., data, cloud resources, etc.)
- Understand how application works and how it can be used

It is meant to complement, not replace, self-paced or instructor-led instruction. For a Systems Engineer, it is mean to be used at this stage in his/her journey.

<img src="https://github.com/cmangubat/app_centric/assets/43074193/b891423f-3c30-4529-b62a-602f24c1c9c7.png" width="500" height="88">

# Baseline application

The application that is at focal point of this framework can be any application that meets the enterprises needs. However, for purposes of this template, the baseline, out-of-the-box, instructions and exercises in this document will leverage the Google-backed microservices-demo application: https://github.com/GoogleCloudPlatform/microservices-demo.

In practice, this application would be either be complemented by other applications that address specific enterprise needs.

**IMPORTANT**: This repository does not include the necessary information and instructions that would be necessary to secure a Cloud Native Application Protection Platform (CNAPP). These must be custom-made for the CNAPP that is paired with the application in this framework.

# Getting started

At a high-level, using the ACE framework involves the following steps:

1. Select a supported Cloud Service Provider (CSP) that will host the application
2. Configure the CSP environment as required by the CNAPP that will protect it
3. Deploy the application to the CSP environment
4. Protect the application with a CNAPP

Out-of-the-box guidance for accomplishing the steps above will be based on the above-mentioned baseline application.



## For AWS

For instructions on how to prepare the environment, see <a href="https://github.com/cmangubat/app_centric/blob/main/csp/aws/aws_env.md">here</a>

## For GCP

For instructions on how to prepare the environment, see <a href="https://github.com/cmangubat/app_centric/blob/main/csp/gcp/gcp_env.md">here</a>

### For Azure

TBA

# Assumptions

This repository makes the following assumptions about the application that will be protected:

- It will be a microserved application based on Kubernetes
- It will be a cloud-based application
