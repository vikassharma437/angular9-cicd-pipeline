# AngularUi

This project was generated with [Angular CLI](https://github.com/angular/angular-cli) version 12.1.0.

## Development server

Run `ng serve` for a dev server. Navigate to `http://localhost:4200/`. The app will automatically reload if you change any of the source files.

## Code scaffolding

Run `ng generate component component-name` to generate a new component. You can also use `ng generate directive|pipe|service|class|guard|interface|enum|module`.

## Build

Run `ng build` to build the project. The build artifacts will be stored in the `dist/` directory.

## Running unit tests

Run `ng test` to execute the unit tests via [Karma](https://karma-runner.github.io).

## Running end-to-end tests

Run `ng e2e` to execute the end-to-end tests via a platform of your choice. To use this command, you need to first add a package that implements end-to-end testing capabilities.

## Further help

To get more help on the Angular CLI use `ng help` or go check out the [Angular CLI Overview and Command Reference](https://angular.io/cli) page.

## Prerequisite for CICD

1. Two Openshift clusters.
2. Install Jenkins on one cluster.
3. Install Nexus Repository Manager on another cluster.
4. Create Dev, SIT and Training namespace on cluster where Nexus is installed.
5. Create UAT and PERF namespace on cluster where Jenkins is Installed.

## Build the Application using Jenkins and Deploy on OpenShift

I have not covered here about how to install Jenkins on Openshift. I simply covered here about how to deploy pipeline using CICD tools like Jenkins.

You can use Jenkinsfile to build and deploy the application on different namespaces like DEV, SIT, UAT, TRAINING and PERF environment.

1. Add Publish Registry in package.json file as specified below. This will allow you to download the dependencies from external urls through Nexus in case your server is running behind the proxy.
   
"publishConfig": {
  "registry": "http://nexus-repository-manager-nexus.apps.cluster-89e8.89e8.sandbox1804.opentlc.com/repository/npm-releases/"
},
  
2. Add Jenkinsfile, sitJenkinsfile and uatJenkinsfile for running the build pipeline on different environments as this is not the continous build pipeline example. We are using here the different Jenkinsfile as per environment.

3. Jenkinsfile will prepare builds, install NPM dependencies and promote the builded image on the specified namespace in Openshift cluster.

4. sitJenkinsfile will be used to copy and tag the DEV namespace builded image to SIT namespace as both the namespaces are in same environment.

5. uatJenkinsfile will be used to promote the image in UAT, PT and TRAINING namespace in all together separate cluster.
