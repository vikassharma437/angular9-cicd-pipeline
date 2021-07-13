#!bash
def appName
def appVersion

pipeline {
  agent {
    label 'nodejs12'
  }
  environment {
    DEV_NAMESPACE = "dev"
    WORKDIR = pwd()
    nexusURL = "http://nexus-repository-manager-nexus.apps.cluster-89e8.89e8.sandbox1804.opentlc.com/repository/npm-releases"
    DEV_API_SERVER = "https://api.cluster-89e8.89e8.sandbox1804.opentlc.com:6443"
    buildconf = "false"
    dc = "false"
    routeProtocol = "HTTPS"
  }
  stages {
    stage ('Prepare Build') {
      steps {
        script {
          appName = sh(script: 'node -e "console.log(require(\'./package.json\').name);"', returnStdout: true).trim()
          appVersion = sh(script: 'node -e "console.log(require(\'./package.json\').version);"', returnStdout: true).trim()
        }
      }
    }
    stage ('Install NPM Dependencies'){
      steps {
        sh '''
        npm cache clean --force
        rm -rf ${WORKDIR}/package-lock.json
        touch .npmrc
        echo "registry=http://nexus-repository-manager-nexus.apps.cluster-89e8.89e8.sandbox1804.opentlc.com/repository/npm-group/" >> .npmrc
        npm install --verbose -d --force
        npm install --save classlist.js --force
      	'''
      }
    }
    stage ('Build Application Package'){
      steps {
        script {
          try {
       		  sh 'PUBLIC_URL=' + appName + '/ npm run build --prod --build-optimizer'
          } catch(e) {
           	print e.getMessage()
            error "error in build app stage"
          }
        }
      }
    }
    stage ('Package the Application'){
      steps {
        script {
          try {
            zip zipFile: "${appName}-${appVersion}.zip", dir: 'dist/angular-ui'
          } catch(e) {
            print e.getMessage()
            error "Error in package app stage"
          }
        }
      }
    }
    stage('Archive the Application') {
      steps {
        script {
          try {
            withCredentials([usernamePassword(credentialsId: 'nexus-credentials', passwordVariable: 'NEXUS_PASSWD', usernameVariable: 'NEXUS_USER')]) {
              sh "curl -u ${NEXUS_USER}:${NEXUS_PASSWD} --upload-file ${WORKDIR}/${appName}-${appVersion}.zip ${nexusURL}/${appName}-${appVersion}.zip"
            }
          } catch(e) {
            print e.getMessage()
            error "Error in archive app stage"
          }
        }
      }
    }
    stage('Create New Build in DEV Namespace') {
  	  steps {
  	    script {
  		    try {
		        withCredentials([usernamePassword(credentialsId: 'dev-ocp-credentials', passwordVariable: 'DEV_OCP_PASSWD', usernameVariable: 'DEV_OCP_USER')]) {
		          echo "Using AppName: ${appName}"
		          sh('oc login -u $DEV_OCP_USER -p $DEV_OCP_PASSWD ${DEV_API_SERVER} -n ${DEV_NAMESPACE} --insecure-skip-tls-verify=true')
              buildconf = sh(script: 'oc get bc ${appName} >> /dev/null 2>&1 && echo "true" || echo "false"', returnStdout: true)
		          buildconf = buildconf.trim()
		          echo "BuildConfig status contains: '${buildconf}'"

		          if(buildconf == 'false') {
              	sh "oc new-build --name=${appName} --image-stream=nginx-118:latest --labels=app=${appName} --binary=true"
              } else {
               	echo "Build Config Already exist."
              }
		        }
          } catch(e) {
            print e.getMessage()
            error "${DEV_NAMESPACE} stage having some issue so this stage can be ignored. Please check logs for more details."
          }
        }
      }
    }
    //Image Deployment
    stage('Deploy Image Build to Dev Namespace') {
      steps {
  	    script {
  	      try {
    		    timeout(time: 600, unit: 'SECONDS') {
    		      withCredentials([usernamePassword(credentialsId: 'dev-ocp-credentials', passwordVariable: 'DEV_OCP_PASSWD', usernameVariable: 'DEV_OCP_USER')]) {
                sh('oc login -u $DEV_OCP_USER -p $DEV_OCP_PASSWD ${DEV_API_SERVER} -n ${DEV_NAMESPACE} --insecure-skip-tls-verify=true')
                sh "oc start-build ${appName} --from-dir=${WORKDIR}/dist/angular-ui/ --wait=true"
                dc = sh(script: 'oc get dc ${appName} >> /dev/null 2>&1 && echo "true" || echo "false"', returnStdout: true)
		            dc = dc.trim()
		            echo "Deployment Config status contains: '${dc}'"

    		        if(dc == 'false') {
                  echo "Deploying the ${appName}:latest application binary to ${DEV_NAMESPACE} namespace"
                  sh """
                  oc new-app ${appName}:latest --as-deployment-config -n ${DEV_NAMESPACE}
                  oc rollout status dc ${appName} -n ${DEV_NAMESPACE} -w
                  oc rollout pause  dc ${appName} -n ${DEV_NAMESPACE}
                  """
                  echo "Application ${appName}:latest deployed successfully to ${DEV_NAMESPACE} namespace."
                } else {
                  echo "Application already exist. Hence rolling out the latest updates on the same."
                  sh """
                  oc rollout pause  dc ${appName} -n ${DEV_NAMESPACE}
                  oc rollout status dc ${appName} -n ${DEV_NAMESPACE} -w
                  oc rollout resume dc ${appName} -n ${DEV_NAMESPACE}
                  """
                  echo "Application ${appName}:latest has been successfully deployed to ${DEV_NAMESPACE} namespace."
                }

                sh "oc set env dc/${appName} TZ=Asia/Kolkata"
                echo "Build ${appName} deployed successfully in ${DEV_NAMESPACE} namespace"
    		      }
    		    }
          } catch(e) {
            print e.getMessage()
        	  error "Build not successful"
          }
	      }
	    }
    }
    stage('Tag Image in Development Project') {
  	  steps {
  	    script {
  	      withCredentials([usernamePassword(credentialsId: 'dev-ocp-credentials', passwordVariable: 'DEV_OCP_PASSWD', usernameVariable: 'DEV_OCP_USER')]) {
              sh('oc login -u $DEV_OCP_USER -p $DEV_OCP_PASSWD ${DEV_API_SERVER} -n ${DEV_NAMESPACE} --insecure-skip-tls-verify=true')
  	        sh "oc tag ${DEV_NAMESPACE}/${appName}:latest ${DEV_NAMESPACE}/${appName}:${env.BUILD_NUMBER} -n ${DEV_NAMESPACE}"
  	      }
  	    }
  	  }
    }
  }
}
