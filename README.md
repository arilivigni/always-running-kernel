# always-running-kernel
This is the sample Openshift project for the always-running-kernel.  Contains the Openshift templates, Dockerfiles, and pipelines

## Structure

    ├── config
    │   ├── Dockerfiles
    │   │   └── skt
    │   │       ├── beaker-client.repo
    │   │       └── Dockerfile
    │   └── s2i
    │       ├── jenkins
    │       │   ├── jenkins-continuous-infra-slave-buildconfig-template.yaml
    │       │   ├── jenkins-persistent-template.yaml
    │       │   ├── master
    │       │   │   ├── configuration
    │       │   │   │   ├── hudson.tasks.Maven.xml
    │       │   │   │   ├── init.groovy
    │       │   │   │   ├── org.jenkinsci.plugins.workflow.libs.GlobalLibraries.xml
    │       │   │   │   └── scriptApproval.xml
    │       │   │   ├── plugins
    │       │   │   │   └── redhat-ci-plugin.hpi
    │       │   │   └── plugins.txt
    │       │   └── slave
    │       │       └── Dockerfile
    │       └── skt
    │           └── skt-template.yml
    ├── Jenkinsfile
    ├── LICENSE
    └── README.md

## Config

The files under this directory contain the Dockerfiles and Jenkins s2i templates

### Jenkins master and slave s2i templates

 * s2i Jenkins master - jenkins-persistent-template.yaml
    * s2i files to create the Jenkins Master are under config/s2i/jenkins/master
    * Openshift Jenkins master template to define building the container in Openshift
 * s2i Jenkins Slave - jenkins-continuous-infra-slave-buildconfig-template.yaml
    * s2i files to create the Jenkins slave are under config/s2i/jenkins/slave/Dockerfile
    * Openshift Jenkins slave template to define building the container in Openshift

### skt s2i template

 * Dockerfile - Setup beaker-client.repo and install packages
 * s2i template - skt-template.yml
    * Openshift skt template to define building the container in Openshift

## Jenkins 2.0 Pipeline - Jenkinsfile
  * Create the podTemplate to use when creating a Kubernetes pod
  * Create the containers that will be run in the pipeline (jnlp not needed but included)
  * Checks out latest master branch skt
  * Runs python ./skt/skt.py --help
  * Can grab logs back from the container if they exist
  * Report sucess or failure
  
## Notes
  * Jenkins master template and config require a persistent volume of 1gi or higher (100gi preferred)
  * Repos and params of containers can be overwritten and changed to match desired location
  

 
 
 
 
 
