def STATUS_COLOR_MAP = [
  "SUCCESS": "good",
  "FAILURE": "danger",
  "UNSTABLE": "danger",
  "ABORTED": "danger"
]

def branchesName() {
  def branchName = "${env.BRANCH_NAME}"

  if (branchName == "uat") {
    return "uat"
  } else if (branchName == "master") {
    return "master"
  } else {
    return "other"
  }
}

def defineRegistry() {
  def branchName = "${env.BRANCH_NAME}"

  if (branchName == "uat") {
    return "uat"
  } else if (branchName == "master") {
    return "prod"
  } else {
    return "other"
  }
}

def defineNamespace() {
  def branchName = "${env.BRANCH_NAME}"

  if (branchName == "UAT") {
    return "uat"
  } else if (branchName == "master") {
    return "prod"
  } else {
    return "other"
  }
}

def defineNodeenv() {
  def branchName = "${env.BRANCH_NAME}"

  if (branchName == "UAT") {
    return "uat"
  } else if (branchName == "master") {
    return "production"
  } else {
    return "other"
  }
}

pipeline {
  environment {
    branch = branchesName()
    registry = defineRegistry()
    namespace = defineNamespace()
    node_env = defineNodeenv()
    service_name = "${env.GIT_URL.replaceFirst(/^.*\/([^\/]+?).git$/, '$1')}"
  }
  agent any
  stages {
    stage('Build Image') {
      when {
        anyOf {
          branch 'uat'
          branch 'master'
        }
      }
      steps {
        script {
          sh "docker build -t ${env.service_name} --no-cache --build-arg namespace=${env.node_env} ."
        }
      }   
    }
    stage('Tag Image') {
      when {
        anyOf {
          branch 'uat'
          branch 'master'
        }
      }
      steps {
        script {
          sh "docker tag ${env.service_name} dubeyg0692/${env.service_name}:latest"
        }
      }      
    }

    stage ('Image Push to docker') {
      when {
        anyOf {
          branch 'uat'
          branch 'master'
        }
      }
      steps {
        script {
          withCredentials([usernamePassword(credentialsId: 'docker-hub-credential', passwordVariable: 'PSW', usernameVariable: 'USR')]) {
            sh 'docker login -u $USR -p $PSW'
            sh "docker push dubeyg0692/${env.service_name}:latest"
          }
        }
      }
    }
  }
}
