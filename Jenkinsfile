pipeline {
  agent {
      label 'maven'
  }
  stages {
    stage('Build App') {
      steps {
        sh "mvn -B install"
      }
    }
    stage('Create Image Builder') {
      when {
        expression {
          openshift.withCluster() {
            return !openshift.selector("bc", "mapit").exists();
          }
        }
      }
      steps {
        script {
          openshift.withCluster() {
            openshift.newBuild("--name=mapit", "--image-stream=redhat-openjdk18-openshift:1.1", "--binary")
          }
        }
      }
    }
    stage('Build Image') {
      steps {
        script {
          openshift.withCluster() {
            openshift.selector("bc", "mapit").startBuild("--from-file=target/mapit-spring.jar", "--wait")
          }
        }
      }
    }
    stage('Setup dev environment') {
      when {
        expression {
          openshift.withCluster() {
            return !openshift.selector("project", "mapit-dev").exists();
          }
        }
      }
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject("mapit-dev") {
              echo "Using project: ${openshift.project()}"
            }
          }
        }
      }
    }
    stage('Promote to DEV') {
      steps {
        script {
          openshift.withCluster() {
            openshift.tag("mapit:latest", "mapit:dev")
          }
        }
      }
    }
    stage('Create DEV') {
      when {
        expression {
          openshift.withCluster() {
            openshift.withProject("mapit-dev") {
              return !openshift.selector('dc', 'mapit').exists()
            }
          }
        }
      }
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject("mapit-dev") {
              openshift.newApp("mapit/mapit:dev", "--name=mapit").narrow('svc').expose()
            }
          }
        }
      }
    }
    stage('Promote STAGE') {
      steps {
        script {
          openshift.withCluster() {
            openshift.tag("mapit:dev", "mapit:stage")
          }
        }
      }
    }
    stage('Create STAGE') {
      when {
        expression {
          openshift.withCluster() {
            return !openshift.selector('dc', 'mapit-stage').exists()
          }
        }
      }
      steps {
        script {
          openshift.withCluster() {
            openshift.newApp("mapit:stage", "--name=mapit-stage").narrow('svc').expose()
          }
        }
      }
    }
  }
}