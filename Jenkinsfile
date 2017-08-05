#!groovy

properties(
    [
        [$class: 'BuildDiscarderProperty', strategy:
          [$class: 'LogRotator', artifactDaysToKeepStr: '14', artifactNumToKeepStr: '5', daysToKeepStr: '30', numToKeepStr: '60']],
        pipelineTriggers(
          [
              pollSCM('H/15 * * * *'),
              cron('@daily'),
          ]
        )
    ]
)
node {

    stage('Checkout') {
        //disable to recycle workspace data to save time/bandwidth
        deleteDir()
        checkout scm
    }

    docker.image('trion/ng-cli-karma:1.2.1').inside {
      stage('NPM Install') {
          withEnv(["NPM_CONFIG_LOGLEVEL=warn"]) {
              sh 'npm install'
          }
      }

      stage('Build') {
          milestone()
          sh 'ng build --prod --aot --sm --progress=false'
      }

      stage('Test') {
          withEnv(["CHROME_BIN=/usr/bin/chromium-browser"]) {
            sh 'ng test --progress=false --watch false'
          }
          junit '**/test-results.xml'
      }

      stage('Lint') {
          sh 'ng lint'
      }
    }
    //end docker

    stage('Archive') {
        sh 'tar -cvzf dist.tar.gz --strip-components=1 dist'
        archive 'dist.tar.gz'
    }

    stage ('Build Docker Image') {
      app = docker.build("demo/angular-app")
    }

    //optionally some acceptance tests for docker image

    stage ('Push Docker Image') {
      milestone()
      docker.withRegistry('http://127.0.0.1:5000/') {
            def tag = "${env.BUILD_NUMBER}".trim()
            app.push(tag)
            app.push("latest")
        }
    }

    stage('Deploy') {
        milestone()
        echo "Deploying..."
    }
}
