# toolform

[![Build Status](https://travis-ci.org/agiledigital/toolform.svg?branch=master)](https://travis-ci.org/agiledigital/toolform)

**Toolform** is a *DevOps* utility for defining deployment environments such that software applications may be developed independently of their eventual deployment environment. This means that developers don't need to be experts in the target tools/platforms given that toolform takes care these concerns via configuration.

Practically speaking it is a command-line tool for software teams to define and configure their continuous integration (build/test) and continuous delivery (deploy) environments consistently and efficiently.

Toolform exists to generate "infrastructure as code" config files for popular target environments (e.g. Jenkins2 build pipelines, spinnaker deployment pipelines, openshift projects, kubernetes environments, minikube, etc.). It works with typical developer workflows to ensure as close as possible representation of "production conditions" in all environments (including developer desktops).

Features
===============================================================================

Key features of Toolform include:

* **Infrastructure as code**: Your source project artefacts, runtimes, topology, and services (both in-house and 3rd party) are described using a high-level configuration syntax. This allows a blueprint of your project and all of its runtime dependencies to be versioned and treated as you would any other code;

* **Target platform agnostic**: Whether targeting a developer's local environment, or setting up the full CI/CD pipeline from bare metal, pluggable backends process the definition to the target tool or platform; and

* **Project agnostic**: Toolform is focussed on the topology, pipeline, and environmental services (such as payment gateways) of deployment environments, meaning it doesn't make restrictions as to the specifics of any particular source project software.

Usage
================================================================================

```
toolform --help

Usage:
    toolform inspect
    toolform generate

Generates deployment configuration from a project definition.

Options and flags:
    --help
        Display this help text.
    --version
        Print the version number and exit.

Subcommands:
    inspect
        Inspect and print the content of the project file
    generate
        Generate config files targeting a particular platform
```

Inspect Command
--------------------------------------------------------------------------------
```
toolform inspect --help

Usage: toolform inspect --input <file>

Inspect and print the content of the project file

Options and flags:
    --help
        Display this help text.
    --input <file>, -i <file>
        Input file
```

Generate Command
--------------------------------------------------------------------------------
```
toolform generate --help

Usage:
    toolform generate minikube
    toolform generate dockercompose
    toolform generate jenkinsfile

Generate config files targeting a particular platform

Options and flags:
    --help
        Display this help text.

Subcommands:
    minikube
        generates config files for Kubernetes (Minikube) container orchestration
    dockercompose
        generates config files for container orchestration
    jenkinsfile
        generates build.Jenkinsfile and deploy.Jenkinsfile for Jenkins Pipeline

```

```
toolform generate minikube --help

Usage: toolform generate minikube --in-file <file> --out-file <file>

generates config files for Kubernetes (Minikube) container orchestration

Options and flags:
    --help
        Display this help text.
    --in-file <file>, -i <file>
        the path to the project config file
    --out-file <file>, -o <file>
        the path to output the generated file(s)

```

```
toolform generate dockercompose --help

Usage: toolform generate dockercompose --in-file <file> --out-file <file>

generates config files for container orchestration

Options and flags:
    --help
        Display this help text.
    --in-file <file>, -i <file>
        the path to the project config file
    --out-file <file>, -o <file>
        the path to output the generated file(s)
```

```
toolform generate jenkinsfile --help

Usage: toolform generate jenkinsfile --in-file <file> --out-file <file> --template-folder-path <folder>

generates Jenkins files for Jenkins to build/deploy the project

Options and flags:
    --help
        Display this help text.
    --in-file <file>, -i <file>
        the path to the project config file
    --out-file <file>, -o <file>
        the path to output the generated file(s)
    --template-folder-path <path>, -t <path>
        the path to the input template file(s)
```

Example Usage
--------------------------------------------------------------------------------

To generate docker compose output for a project, run:
`toolform generate dockercompose -i your-project.conf -o ./target-dir/target-file.yaml`

This will generate a docker compose file at the specified path.

Jenkins Pipeline File Template
--------------------------------------------------------------------------------

To generate Jenkins Pipeline script files, templates need to be provided. It accepts only Mustache template. Current templates are provided in `/templates` folder.
There are 2 templates `build.Jenkinsfile.mustache` and `deploy.Jenkinsfile.mustache`, each accepts different mappings. Available mappings:

```
// build.Jenkinsfile.mustache
{{projectName}} -> name of the project, typically extracted from the project id of the component in project.conf.
{{label}}       -> project build pod label.
{{builds}}      -> building components of the projects that will is produced based on the component in project.conf.
{{libraries}}   -> Jenkins libraries that is produced based on the builders used in project.conf.
{{volumes}}     -> volumes that are required for the builders.
{{containers}}  -> containers definitions to run the build.
```

```
// deploy.Jenkinsfile.mustache
{{projectName}} -> name of the project, typically extracted from the project id of the component in project.conf.
{{components}}  -> components that will be deployed based on the deployment spec, e.g. OpenShift deploy spec.
```

Project Configuration
================================================================================

A toolform project is defined in [HOCON](https://github.com/lightbend/config/blob/master/HOCON.md) syntax. The configuration is usually split into three separate files: `environment.conf`, `project.conf` and `topology.conf`. The command line tool only accepts a single file as an input but the HOCON format allows for a file to include any number of other files.

environment.conf
--------------------------------------------------------------------------------

This file specifies configuration values that get [substituted](https://github.com/lightbend/config/blob/master/HOCON.md#substitutions) into the other config files using path expressions.
HOCON also supports environment variable subsititution so you can provide a default value that can be overridden by an environment variable.
For example, in the following code snippet  the default port of the API server is set to 8080 but it can also be overwriten by the `API_SERVER_PORT` environment variable.

```
environment.component.api_server.port = 8080
environment.component.api_server.port = ${?API_SERVER_PORT}
```

At the end of this file you can include the other project configuration files like so

```
include "project.conf"
include "topology.conf"
```

project.conf
--------------------------------------------------------------------------------

This file defines all the components and resources that make up a project.

A component is a service that is part of the project, for example, an API server.
A resource is a service that is usually provided by a third party, for example, a Postgres database server.

topology.conf
--------------------------------------------------------------------------------

This file defines how components and resources are connected together and how services are exposed to the outside world.

A link is currently only informational, and defines a link between two services.

An endpoint defines how services and components are exposed outside the cluster.
Endpoints are manifested differently depending on the platform. For example, the OpenShift platform uses routes.
Endpoints are unsupported by some platforms. For example, in Minikube you usually expose services by making them a NodePort and accessing them directly.

Sample Project
--------------------------------------------------------------------------------

The best place to start understanding the project definition format is to look at the example in
src/test/resources/testprojects/realworldsample, starting with the environment.conf file.

Generating a Standalone Launcher
================================================================================

A self contained executable can be generated from the Toolform maven package using [Coursier](https://github.com/coursier/coursier). This is useful to avoid managing dependencies when distributing the tool and allows you to easily install Toolform  in common locations for executables such as /usr/local/bin.

The following steps outline how to generate a standalone launcher.

1. Build Toolform and publish to your local repo.
 
    `$ sbt publishLocal`

2. [Download and install Coursier](https://github.com/coursier/coursier#command-line)
3. Run the bootstap script located at [/scripts/generate-launcher.sh](https://github.com/agiledigital/toolform/blob/master/scripts/generate-launcher.sh).

    `$ scripts/generate-launcher.sh`
 
4. The toolform standalone binary should now be located in the root of your repo.

    ```
    $ ./toolform
    Missing expected command (inspect or generate)!
    
    Usage:
        toolform inspect
        toolform generate
    ...

Creating a fresh jenkins installation and using config maps generated from toolform
========================================================================================

Install a fresh Jenkins and configure it

1. Install helm  
 For Windows powershell  
    `choco install kubernetes-helm`  
 For ubuntu  
    `snap install helm`  
 For windows subsystem for linux  
    `$ curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 > get_helm.sh`  
    `$ chmod 700 get_helm.sh`  
    `$ ./get_helm.sh`  

2. Then add the stable chart repo to helm (https://kubernetes-charts.storage.googleapis.com/)  

    `helm repo add stable`
    
3. Run the following command to set up initial basic configuration for jenkins. Replace `$whoami` with custom name if you want a custom namespace.
    `helm template $(whoami) stable/jenkins --set namespaceOverride=$(whoami)-jenkins --set master.testEnabled=false --set master.ingress.enabled=true --set master.ingress.hostName=jenkins.local  | kubectl apply -f -`
 
4. Now let's get the password for jenkins  

    `kubectl get secrets $(whoami)-jenkins -n $(whoami)-jenkins -o=yaml | grep jenkins-admin-password:| tr -s " " | cut -d ' ' -f 3 | base64 --decode`  

5. Port forward to localhost using the following command  

    `kubectl port-forward $(kubectl get pods -n $(whoami)-jenkins | grep $(whoami)-jenkins | cut -f 1 -d ' ' ) 8080:8080 -n $(whoami)-jenkins`  
    If using windows powershell, the above command will not work as expected. In that case, seperately get the name of the pod using the following command  
    `kubectl get pods -n 'namespace'`  
    Then use the following command to port forward  
    `kubectl port-forward -n namespace podname 8081:8080`
    
The commands above will install jenkins with basic configurations. Now it is time to add customised configurations  

1. Log into jenkins. It would take you directly to Manage Jenkins page if logging in for the first time. Click on Configure System  
2. Navigate to Jenkins Location. Fill in the System Admin e-mail address.
3. Navigate to Global Pipeline Libraries and click on Add to add a library required by the project. The libraries specified at this stage will be global libraries accessible to all the project folders on this particular jenkins installation. There is also option to define libraries within the scope of a folder (project)  
4. In order to add a pipeline library from github or bitbucket, the relevant plugin needs to be enabled. Click on Jenkins home icon and then on the left bar, click on Manage Jenkins. Then on the page that comes up, click on Manage Plugins. Navigate to the Available tab and then search for github. Click and install github authentication plugin from the list of plugin that comes up. For bitbucket, search and install Bitbucket OAuth. The following plugins are commonly used with these pipelines  
    
    `cloudbees-bitbucket-branch-source:2.6.0`  
    `github-branch-source:2.5.8`  
    `slack:2.34`  
    `ansicolor:0.6.2`  
    `config-file-provider:3.6.2`  
    `cobertura:1.15`

5. Credentials need to be entered separately in order to be able to access these repos. To add a credential, go to Jenkins home page and then click on Credentials on the left bar. Then click on System and on the new page that comes up, click on Global credentials. This new page will display all the existing credentials (if any). On the left bar, click on Add Credentials. A form will come up, fill it out with relevant credential details and click ok. This will add the credential and make it accessible on the Source code management section of the Configure page.  
6. 