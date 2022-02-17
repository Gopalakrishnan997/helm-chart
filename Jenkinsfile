pipeline {
	agent any

	parameters {
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
		string(
				name: 'imageVersion',
				description: 'The specified version of the Docker image should be available in artifactory. ' +
								'Default is \'latest\' available docker image. ',
				defaultValue: 'latest',
				trim: true
		)
	}
	stages {
			stage('Check Helmchart config') {
			 when { changeset "helm/*"}
            steps {
				def chart = readYaml (file: 'helm/Chart.yaml')
                def chartVersion = chart.version.toString()
				 sh "helm lint ${WORKSPACE}/helm"
        }
    stage("Release helmchart") {
            steps {
			when { changeset "helm/*"}
               script {
					   echo "Helmchart version  - ${chartVersion}"
					   echo "App version - " + getVersion()
                       sh "helm package ${WORKSPACE}/helm --version ${chartVersion} --app-version ${imageVersion}"
                       rtUpload(
                       serverId: 'jfroginstance',
                       spec: '''{
                           "files": [{
                           "pattern": "*.tgz",
                           "target": "${Service}"
                           }]
                        }
                        '''
                       )
                }
            }
         
        }
		stage('Update repo and fetch package') {
            steps {
			sh "helm repo update"
            sh "helm fetch ${SERVICE}/helm --version ${chartVersion}"
            }
        }
    stage('Untar helmchart package') {
            steps {
                 sh "mv ${WORKSPACE}/helm-${chartVersion}.tgz ${WORKSPACE}/helm-${chartVersion}.tar.gz"            
                 sh "tar xf ${WORKSPACE}/helm-${chartVersion}.tar.gz"
            }
        }
    stage('deploy helmchart') {
            steps {
            if (params.Environment == 'DEV') {
            def values = "values-dev.yaml"
			nameSpace = "fromi-user-migration-dev"
            } else if (params.Environment == 'QA') {
            def values = "values-qa.yaml"
			nameSpace = "fromi-user-migration-qa"
			}
            sh "helm upgrade --install ${SERVICE} ${SERVICE}/helm -f ${WORKSPACE}/helm/${values} --namespace ${nameSpace} --version ${chartVersion} --set image.tag=${imageVersion}"
            }
        }    
}
