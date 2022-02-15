pipeline {
  agent any
  stages {
    stage('build-Vote') {
      agent {
        docker {
          image 'python:2.7.16-slim'
          args '--user root'
        }

      }
      steps {
        echo 'Compiling Vote app'
        dir(path: 'vote') {
          sh 'pip install -r requirements.txt'
        }

      }
    }

    stage('docker-Package-vote') {
      agent any
      steps {
        echo 'Running unit test on vote App'
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'test') {

            def voteImage = docker.build("aderock/vote:v${env.BUILD_ID}", "./vote")
            voteImage.push()
            voteImage.push("${env.BRANCH_NAME}")



          }
        }

      }
    }

    stage('build-worker') {
      agent {
        docker {
          image 'maven:amazoncorretto'
          args '-v $HOME/.m2:/root/.m2'
        }

      }
      steps {
        echo 'Compliling worker app'
        dir(path: 'worker') {
          sh 'mvn compile'
        }

      }
    }

    stage('test-Worker-App') {
      agent {
        docker {
          image 'maven:amazoncorretto'
          args '-v $HOME/.m2:/root/.m2'
        }

      }
      steps {
        echo 'Running unit test on worker App'
      }
    }

    stage('package-worker-app') {
      agent {
        docker {
          image 'maven:amazoncorretto'
          args '-v $HOME/.m2:/root/.m2'
        }

      }
      steps {
        echo 'Packaging worker App'
        dir(path: 'worker') {
          sh 'mvn package -Dskiptest'
          archiveArtifacts(artifacts: '**/target/*.jar', fingerprint: true)
        }

      }
    }

    stage('docker-Package-Worker') {
      agent any
      steps {
        echo 'Running unit test on worker App'
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'test') {

            def workerImage = docker.build("aderock/worker:v${env.BUILD_ID}", "./worker")
            workerImage.push()
            workerImage.push("${env.BRANCH_NAME}")



          }
        }

      }
    }

    stage('build-result-app') {
      agent {
        docker {
          image 'node:erbium-stretch'
        }

      }
      steps {
        echo 'Compiling result app'
        dir(path: 'result') {
          sh 'npm install'
        }

      }
    }

    stage('Test-result') {
      agent {
        docker {
          image 'node:erbium-stretch'
        }

      }
      steps {
        echo 'Compiling result app'
        dir(path: 'result') {
          sh 'npm install'
          sh 'npm test'
        }

      }
    }

    stage('docker-Package-Result') {
      agent any
      steps {
        echo 'Running unit test on result App'
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'test') {

            def resultImage = docker.build("aderock/result:v${env.BUILD_ID}", "./result")
            resultImage.push()
            resultImage.push("${env.BRANCH_NAME}")



          }
        }

      }
    }
      stage('Sonarqube') {
      agent any
      /*when{
        branch 'master'
      }*/
      tools {
        jdk "JDK11" // the name you have given the JDK installation in Global Tool Configuration
      }

      environment{
        sonarpath = tool 'SONAQUBE'
      }

      steps {
            echo 'Running Sonarqube Analysis..'
            withSonarQubeEnv('sonar-instavote') {
              sh "${sonarpath}/bin/sonar-scanner -Dproject.settings=sonar-project.properties -Dorg.jenkinsci.plugins.durabletask.BourneShellScript.HEARTBEAT_CHECK_INTERVAL=86400"
            }
      }
    }


    stage("Quality Gate") {
        steps {
            timeout(time: 1, unit: 'HOURS') {
                // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                // true = set pipeline to UNSTABLE, false = don't
                waitForQualityGate abortPipeline: true
            }
        }
    }

    stage('deploy to dev ') {
      agent any
      steps {
        sh 'docker-compose up -d'
      }
    }

  }
}