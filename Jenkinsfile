pipeline {
  agent any
  environment {
        docker_username = 'horib'
  }
  stages {
    stage('clone down') {
      steps {
        stash excludes: '.git', name: 'code'
      }
    }
    stage('Parallel execution') {
      parallel {
        stage('Say Hello') {
          steps {
            sh 'echo "hello world"'
          }
        }

        stage('build app') {
          options {
            skipDefaultCheckout(true) }
          agent {
            docker {
              image 'gradle:jdk11'
            }
          }
          steps {
            unstash 'code'
            sh 'ci/build-app.sh'
            archiveArtifacts 'app/build/libs/'
            stash excludes: '.git', name: 'code'
            sh 'ls -lah'
            deleteDir()
            sh 'ls '
          }
        }
        stage('test app') {
          options {
            skipDefaultCheckout(true) }
          agent {
            docker {
              image 'gradle:jdk11'
            }
          }
          steps {
            unstash 'code'
            sh 'ci/unit-test-app.sh'
            junit 'app/build/test-results/test/TEST-*.xml'
          }
        }
      }
    }
    stage('push docker app') {
      when {
        beforeAgent true
        branch 'master'
      }
      environment {
          DOCKERCREDS = credentials('docker_login') //use the credentials just created in this stage
        }
        options {
            skipDefaultCheckout(true) 
            }
        steps {
          unstash 'code' //unstash the repository code
          sh 'ci/build-docker.sh'
          sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin' //login to docker hub with the credentials above
          sh 'ci/push-docker.sh'
      }
    }
    stage('component test'){
      when {
        anyOf{
          beforeAgent true
          branch 'master'
          changeset 'master'
        }
      }
      options{
        skipDefaultCheckout(true)
      }
      steps{
        unstash 'code'
        sh 'ci/component-test.sh'
      }
    }
  }
}
void pushIfMaster() {
    if (BRANCH_NAME=="master"){
      sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin'
      sh 'ci/push-docker.sh'
    }

}
