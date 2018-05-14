Topics
=================

  * [icm-jenkins-common](#icm-jenkins-common)
  * [Configure Shared Library](#configure-shared-library)
    * [Configure the shared library at global level](#configure-the-shared-library-at-global-level)
    * [Configure the shared library at Organization level](#configure-the-shared-library-at-Organization-level)
  * [How to use Shared Library in Jenkinsfile](#how-to-use-shared-library-in-jenkinsfile)
    * [Load Shared Library Implicitly](#load-shared-library-implicitly)
    * [Load Shared Library Explicitly](#load-shared-library-explicitly)
    
# icm-jenkins-common

The Jenkins Pipeline has support for creating "Shared Libraries" which can be defined in external source control repositories and loaded into existing Pipelines. `icm-jenkins-common` is a common jenkins library repository which can be used across multiple projects.

This library contains common utility functions which can be used for multiple projects. For example, this library contains custom function for docker stages which can be used across multiple projects to `build` and later `tag and push` the docker image to particular container registry.

See the [directory structure](https://jenkins.io/doc/book/pipeline/shared-libraries/) for more information about the shared libraries structure.

icm-jenkins-common library has mainly two folders `src/` and the `vars/`. The `vars/` folder has all the stages and the steps which can be used individually by calling them or grouping different steps to form a stage. There are some common stages available in the `vars/` which can be used directly according to the need. For example, `icmDeployWithHelmStages` stage function has the couple of steps grouped together to setup the `kubeconfig` for the specified cluster and to `update/install` the provided `helm chart` for the specified cluster. The functionality for most of these functions in the `vars/` is present within the Groovy classes of `src/` folder.

## Configure Shared Library

The shared library can be configured at different levels in Jenkins depending on the use-case. We can configure the shared library at global level, at the organization level and at the folder level. This can be configured once the plugin [Pipeline Shared Groovy Libraries Plugin](https://wiki.jenkins.io/display/JENKINS/Pipeline+Shared+Groovy+Libraries+Plugin) is installed in the Jenkins. 


### [Configure the shared library at global level](https://jenkins.io/doc/book/pipeline/shared-libraries/#global-shared-libraries)  

We can configure the shared library at the global jenkins level from `Manage Jenkins » Configure System » Global Pipeline Libraries` and we can use as many libraries as necessary in the pipeline.

<img src="https://github.ibm.com/TheWeatherCompany/icm-jenkins-common/blob/pipeline-test/images/configure_shared_library.png" width="800">


### Configure the shared library at Organization level
 
At the organization level the shared libraries can be configured from `Jenkins » Organization » Configure » Pipeline Libraries`.

<img src="https://github.ibm.com/TheWeatherCompany/icm-jenkins-common/blob/pipeline-test/images/project_pipeline_configuration.png" width="800">

Any Project created can have Shared Libraries associated with it. This mechanism allows scoping of specific libraries to all the Pipelines inside of the project or sub-project.



## How to use Shared Library in Jenkinsfile

There are two ways to use shared library configured in the Jenkins.

### Load Shared Library Implicitly 

Shared Libraries marked Load implicitly allows Pipelines to immediately use classes or global variables defined by any such libraries. To access other shared libraries, the `Jenkinsfile` needs to use the `@Library` annotation, specifying the library’s name:

<img src="https://github.ibm.com/TheWeatherCompany/icm-jenkins-common/blob/pipeline-test/images/load_implicitly.png" width="800">

    
### Load Shared Library Explicitly

We need to use `@Library` annotation in `Jenkinsfile` as mentioned below to load shared library explicitly. By default it will checkout the shared library from the configured branch.

```
@Library('my-shared-library')
```


User can also load shared library from any branch specifying the `@branch-name`.

```
@Library('my-shared-library@branch-name')
```

We can also define the multiple shared libraries in the single `Jenkinsfile` as below:

```
@Library(['my-shared-library1', 'my-shared-library2'])
```
