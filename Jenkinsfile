node {
    def DOCKER_HOST
    stage('Preparation') { 
        // Get some code from a GitHub repository
        git 'https://github.com/ivanivanoff/test_jenkins.git'
      
        // Prepare the env vars
        // default value is tcp://docker:2376
        // DOCKER_HOST = 'docker-host'
    }
    stage('Build') {
        withEnv(["DOCKER_HOST=$DOCKER_HOST"]) {
          try {
            sh "docker-compose -f docker-compose-test.yml build test"
            sh "docker-compose -f docker-compose-test.yml run test"
          } finally {
            sh "docker-compose -f docker-compose-test.yml down -v"
          }
        }
    }
    stage('Publish') {
     if (env.BRANCH_NAME == "master") {
        stage('Publish') {
          withCredentials([
            string(
              credentialsId: 'aws_account_id',
              variable: 'aws_account_id'
            )
          ]) {
            def awsRegistry = "${env.aws_account_id}.dkr.ecr.eu-central-1.amazonaws.com"
            docker.withRegistry("https://${awsRegistry}", "ecr:eu-central-1:ecr-credentials") {
              sh "docker build \
                -t ${awsRegistry}/simple-server:${env.BRANCH_NAME} \
                -t ${awsRegistry}/simple-server:${scmVars.GIT_COMMIT} ."
              sh "docker push ${awsRegistry}/simple-server:${env.BRANCH_NAME}"
              sh "docker push ${awsRegistry}/simple-server:${scmVars.GIT_COMMIT}"
            }
          }
        }
      }
    }
}
