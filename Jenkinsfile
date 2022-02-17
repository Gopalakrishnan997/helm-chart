def VERSION = "1.3.16"
def APPVERSION = "1.3.15"
def PATH = "/var/lib/jenkins/jobs/CI/workspace"
pipeline {
  agent any
  stages {
    stage('cleaning Jenkins workspace') {
      steps {
        deleteDir()
      }
    }

    stage('helm-lint') {
    when { changeset "oc-deployment/**"}
      steps {
        sh "helm lint ${WORKSPACE}/oc-deployment"
        echo "helm lint success"
      }
      post {
        success {
          echo "helm lint success"
        }

      }
    }
     stage('check change of version') {
    when { changeset "oc-deployment/**"}
      steps {
        script {
        sh "helm fetch default-helm/oc-deployment "
        files = findFiles(glob: '*.tgz*')
        def latestVersion = getLatestVersion(files[0].name)
        echo "latest version is-${latestVersion}"
      }
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
