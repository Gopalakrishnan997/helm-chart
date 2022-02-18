pipeline {
    agent any
    parameters {
        	booleanParam(name: 'TriggerDeployment', defaultValue: false,
				description: 'If this parameter is enabled, the pipeline will trigger the helmchart deployment with below parameters ' +
						'If this parameter is not enabled, the pileline will check for the changes in helmchart and trigger the deployment with new chart version.')
		choice(
				name: 'Environment',
				description: 'The environment to which to deploy',
				choices: ['DEV', 'QA']
		)
		choice(
				name: 'Service',
				description: 'The service to be deployed',
				choices: ['fromi-otp-service', 'fromi-pwd-service', 'fromi-usermigration-service', 'fromi-domain-service']
		)
        choice(
				name: 'VersionAutoIncrement',
				description: 'The helmchart version autoincrement',
				choices: ['PATCH', 'MINOR', 'MAJOR' ]
		)
		string(
				name: 'imageAppVersion',
				description: 'The specified version of the Docker image should be available in artifactory. ' +
								'Default is \'null\' available docker image. ',
				defaultValue: '1.0.1',
				trim: true
		)
        string(
				name: 'helmVersion',
				description: 'The specified version of the helmchart version  should be available in artifactory. ' +
								'Default is \'null\' available helmchart version. ',
				defaultValue: '',
				trim: true
		)
	}
    environment {
		JENKINS_WEBHOOK_DEPLOYMENT_TOKEN = 'FROMI_Deployment_gkaiwcxsd'
		REPO = 'default'
    }
    stages {
        	stage("Git checkout") {
	   	 steps {
          git credentialsId: '13d412be-0d91-447a-b125-db7f5800cdb2', url: 'https://github.com/Gopalakrishnan997/helm-chart.git'
         }
        }
stage("Check Helmchart config") {
         when { changeset "oc-deployment/**"}
           steps {
               
             sh "helm lint ${WORKSPACE}/oc-deployment"
             //check for helmchart syntax errors
             echo "helm lint success"
           }
        }
		stage("Release helmchart") {
         when { changeset "oc-deployment/**"
                expression { 
                 params.TriggerDeployment == false
                }
         }
           steps {
            script{
             sh "helm fetch default-helm/oc-deployment"
             //fetch latest version of helmchart from artifactory
             files = findFiles(glob: '*.tgz*')
             def latestVersion = getLatestVersion(files[0].name)
             //Fetch helmchart version from helmchart package
             echo "LatestVersion is-${latestVersion}"
             def newVersion = getNextVersion(latestVersion)
             //Autoincrement to new release version 
             echo "NextVersion is-"+getNextVersion(latestVersion)
             sh "helm package ${WORKSPACE}/oc-deployment --version ${newVersion} --app-version ${IMAGEAPPVERSION}"
             //Package helmchart with new version
             rtUpload(
             serverId: 'jfroginstance',
             spec: '''{
             "files": [{
             "pattern": "*.tgz",
             "target": "${REPO}-helm"
              }]
              }
              '''
              //push helmchart to artifactory
              )
             // sh "curl --fail -X POST -H \"Content-Type: application/json\" --data \"{'service':'${params.Service}','imageVersion':'${params.imageVersion}','helmchartVersion':${newVersion} 'environment':'${params.Environment}'}\" https://jenkins-value-added-services-dev.v11.np01.ocp.six-group.net/generic-webhook-trigger/invoke?token=${JENKINS_WEBHOOK_DEPLOYMENT_TOKEN}"
              //trigger helmchart deployment
           }
           }
		}
       stage('Trigger Deployment') {
        when { 
            expression {
                return params.TriggerDeployment
            }
        }
        steps {
           // sh "curl --fail -X POST -H \"Content-Type: application/json\" --data \"{'service':'${params.Service}','imageVersion':'${params.imageVersion}','helmchartVersion':${params.helmVersion} 'environment':'${params.Environment}'}\" https://jenkins-value-added-services-dev.v11.np01.ocp.six-group.net/generic-webhook-trigger/invoke?token=${JENKINS_WEBHOOK_DEPLOYMENT_TOKEN}"
            echo "Trigger Deployment"
        }
    }   
    }
}
def getLatestVersion(service) {
   
      if (service == null || service.length() == 0) {
        return service;
    }
    return service.substring(service.indexOf('.')-1,service.length()-4);
}

def getNextVersion(latestVersion) {
 def (major, minor, patch) = latestVersion.tokenize('.').collect { it.toInteger() }
if(params.VersionAutoIncrement == "MAJOR"){
     return "${major + 1}.0.0";
  }
 else if(params.VersionAutoIncrement == "MINOR"){
     return "${major}.${minor + 1}.0";
  }
  else{
      return "${major}.${minor}.${patch + 1}";
  }
}
