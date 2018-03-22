# This repo is currently in work and changing hourly - I'll remove this warning when it's stable
## 2018-03-22

# OpenShift Examples - CI/CD Pipeline
OpenShift can be a useful aide in creating a Continuous Integration (CI) / Continuous Delivery (CD) pipeline.  CI/CD is all about creating a streamlined process to move from a developer's code change to delivered operations-ready software (i.e. ready to deploy to production).  And a key part of CI/CD is the automation to make the process predicatable, repeatable, and easy.

This git repo contains an intentionally simple example of a software pipeline to deploy a webapp. And it showcases the tight intergration between Jenkins and OpenShift.  Namely the multiple plugins that enable shared auth, preform synchronization between Jenkins and OpenShift, allow for steps to be written in a readable and comprehensive syntax (DSL).

When you kickoff this example, the flow is as follows: perform pre-build -> do an app source code build + containerization -> do some automated testing -> deploy the app to the dev environment -> tag an image so that external image stream triggers can pull the image into a staging environment.  

Here's what it looks like:

![Screenshot](./.screens/ocppipeline.gif)

###### :information_source: This example is based on OpenShift Container Platform version 3.7.  It could work with older versions but has not been tested.


## How to put this in my cluster?
First off, you need access to an OpenShift cluster.  Don't have an OpenShift cluster?  That's OK, download the CDK for free here: [https://developers.redhat.com/products/cdk/overview/][1]

There is a script you can use for creating all the projects and required components for this example.

 > `pipeline_setup.sh`

Because no other Jenkins server is already configured for use, OpenShift will actually create one for use within this project.  We will wait until that jenkins server is ready - you can see status with `oc get pods`.

Once it's ready you can kick off a new pipeline build via the CLI or the web console.

The CLI command is:
> `oc start-build -F openshiftexamples-cicdpipeline`


## Why pipelines?
The most obvious benefits of CI/CD pipelines are:
* Deliver software more efficiently and rapidly
* Free up developer's time from manual build/release processes
* Standardize a process that requires testing before release
* Track the success and efficiency of releases plus gain insight into (and control over) each step


## About the code / software architecture
The parts in action here are:
* A sample web app
* OpenShift Jenkins [server image](https://github.com/openshift/jenkins#installation)
	* Includes the [OpenShift client plugin](https://github.com/openshift/jenkins-client-plugin) (newer OpenShift installs)
	* Includes the [OpenShift Jenkins plugin](https://github.com/openshift/jenkins-plugin) (older OpenShift installs)
* OpenShift Jenkins [slave images](https://access.redhat.com/containers/#/search/jenkins%2520slave)
* A Jenkinsfile (using OpenShift DSL)
* Instant app template YAML file (to create/configure everything easily)
* Key platform components that enable this example
	* Projects and Role Based Access Control (RBAC)
	* Integration with Jenkins
	* Source code building via s2i
	* Container building via BuildConfigs
	* Deployments via [image change triggers][3]


## Other considerations
The Jenkins integration can come in a varitey of different flavors. See below for some disucssion on things to consider when doing this for your projects.
* Where does Jenkins get deployed? e.g. in each project, shared in global project, external to OpenShift
	* Openshift can autoprovision Jenkins
	* You can pre-setup Jenkins to be shared by multiple projects
	* If you've already got a Jenkins server and you just want to hook into it
* How does the pipeline move images through dev/test/prod?
	* triggers to pull?
	* project to project?
	* via tagging images?
* Can everyone do anything with Jenkins or will you define roles and SCC/RoleBindinds to restrict actions?
* Will you use [slave builders][4]?
* Where is the Jenkinsfile?
	* in git - easy to manage independently
	* embeded in an OpenShift BuildConfig template - doesn't reqire a git fetch, editable within OpenShift
* What OpenShift integration hooks will you use?
* Does production have a separate cluster?
* Do you want to roll your own Jenkins image?
	* The images that come with OpenShift are tested to work - if you roll your own to make sure any plugins used align with the platform version
	* You can also [override the jenkins image with s2i][2]
* Coordinating individual microservice builds and running integration tests
* Performing coverage tests
* Doing code scanning


## References and other links to check out (trying to keep these in a best-first order)
* https://github.com/openshift/jenkins-client-plugin
* https://blog.openshift.com/building-declarative-pipelines-openshift-dsl-plugin/
* https://github.com/OpenShiftDemos/openshift-cd-demo
* https://docs.openshift.com/container-platform/3.7/using_images/other_images/jenkins.html
* https://docs.openshift.com/container-platform/3.7/dev_guide/dev_tutorials/openshift_pipeline.html
* https://docs.openshift.com/container-platform/3.7/install_config/configuring_pipeline_execution.html
* https://blog.openshift.com/cross-cluster-image-promotion-techniques/
* https://blog.openshift.com/openshift-pipelines-jenkins-blue-ocean/


## License
Under the terms of the MIT.

[1]: https://developers.redhat.com/products/cdk/overview/
[2]: https://docs.openshift.com/container-platform/3.7/using_images/other_images/jenkins.html#jenkins-as-s2i-builder
[3]: https://docs.openshift.com/container-platform/3.7/dev_guide/builds/triggering_builds.html#image-change-triggers
[4]: https://docs.openshift.com/container-platform/3.7/using_images/other_images/jenkins.html#using-the-jenkins-kubernetes-plug-in-to-run-jobs
[5]: http://v1.uncontained.io/playbooks/continuous_delivery/external-jenkins-integration.html
[6]: https://github.com/snowdrop/cloud-native-backend/tree/master/openshift
