def STATUS_COLOR_MAP = [
  "SUCCESS": "good",
  "FAILURE": "danger",
  "UNSTABLE": "danger",
  "ABORTED": "danger"
]

def branchesName() {
  def branchName = "${env.BRANCH_NAME}".toLowerCase()

  switch (branchName) {
    case "uat":
      return "uat"
    case "master":
      return "master"
    default:
      return "other"
  }
}

def defineRegistry() {
  def branchName = "${env.BRANCH_NAME}".toLowerCase()

  switch (branchName) {
    case "uat":
      return "uat"
    case "master":
      return "prod"
    default:
      return "other"
  }
}

def defineNamespace() {
  def branchName = "${env.BRANCH_NAME}".toUpperCase()

  switch (branchName) {
    case "UAT":
      return "uat"
    case "MASTER":
      return "prod"
    default:
      return "other"
  }
}

def defineNodeenv() {
  def branchName = "${env.BRANCH_NAME}".toUpperCase()

  switch (branchName) {
    case "UAT":
      return "uat"
    case "MASTER":
      return "production"
    default:
      return "other"
  }
}

pipeline {
  agent any
  environment {
    branch = branchesName()
    registry = defineRegistry()
    namespace = defineNamespace()
    node_env = defineNodeenv()
    service_name = "${env.GIT_URL.replaceFirst(/^.*\/([^\/]+?).git$/, '$1')}"
  }
  stages {
    stage('Build Image') {
      when {
        expression { branch in ["uat", "master"] }
      }
      steps {
        script {
          sh "docker build -t ${env.service_name} --no-cache --build-arg namespace=${env.node_env} ."
        }
      }
    }
    stage('Tag Image') {
      when {
        expression { branch in ["uat", "master"] }
      }
      steps {
        script {
          sh "docker tag ${env.service_name} dubeyg0692/${env.namespace}/${env.service_name}:${BUILD_NUMBER}"
        }
      }
    }
    stage('Image Push to docker') {
      when {
        expression { branch in ["uat", "master"] }
      }
      steps {
        script {
          withCredentials([usernamePassword(credentialsId: 'docker-hub-credential', passwordVariable: 'PSW', usernameVariable: 'USR')]) {
            sh 'docker login -u $USR -p $PSW hub.docker.com'
            sh "docker push dubeyg0692/${env.namespace}/${env.service_name}:${BUILD_NUMBER}"
          }
        }
      }
    }
    stage('Update Image to latest') {
      when {
        expression { branch in ["uat", "master"] }
      }
      steps {
        script {
          withCredentials([usernamePassword(credentialsId: 'git-hub-credential', passwordVariable: 'PSW', usernameVariable: 'USR')]) {
            sh "git config user.email gaurav.dubey@nagarro.com"
            sh "git config user.name $USR"
            sh 'rm -rf argo'
            sh "git clone https://$USR:$PSW@git@github.com:gorakhgaurav/argo.git"
            dir('argo') {
              sh "git checkout ${env.branch}"
              sh "sed -i 's/{{ .Chart.Name }}:[^ ]*/{{ .Chart.Name }}:${BUILD_NUMBER}/g' ${env.service_name}/templates/deployment.yaml"
              sh "git add ${env.service_name}/templates/deployment.yaml"
              sh "git commit -m 'Update image version to: ${BUILD_NUMBER}'"
              sh "git push https://$USR:$PSW@git@github.com:gorakhgaurav/argo.git HEAD:${env.branch}"
            }
          }
        }
      }
    }
  }
}
