pipeline{
    parameters {
        booleanParam(name: 'del_cache', defaultValue: false, description: 'Toggle this value to build with cache clearing.')
    }
    agent any
    environment {
        SLACK_CHANNEL = '#maintenance-test'
        IMAGE_REPO = 'XXXXXXXXXXXX.dkr.ecr.eu-west-1.amazonaws.com/artem'
        KUBECTL = 'kubectl --kubeconfig=/var/lib/jenkins/.kube/k8s-artem.yml'
        PROJECT_NAME = 'test-web'
        REPO = 'https://git.artem.services/scm/dev/test-web.git'
        REPO_CRED = 'svc-bitbucket'
    }
    options {
        ansiColor('xterm')
        timeout(time: 30, unit:'MINUTES')
        timestamps()
    }
    stages {
        stage ('Authorization in repo') {
            steps {
                script {
                    sh 'eval "$(aws ecr get-login --profile artem --region eu-west-1 --no-include-email)" &> /dev/null'
                }
            }
        }
        stage ('Delete Work Dir') {
            when { expression { return params.del_cache } }
            steps {
                deleteDir()
                git branch: "${BRANCH_NAME}", credentialsId: "${REPO_CRED}", url: "${REPO}"
            }
        }
        stage ('Build app') {
            agent {
                docker {
                    image 'node:latest'
                    args "--name ${BUILD_TAG}-app \
                    -v /etc/passwd:/etc/passwd:ro"
                    reuseNode true               
                }
            }
            environment {
                HOME = "${WORKSPACE}"
            }
            steps {
                sh "npm install"
                sh "npm run build"
            }
        }
        stage ('Build app image') {
            steps {
                script {
                    withDockerServer([uri:'tcp://127.0.0.1:4243']) {
                        sh 'eval "$(aws ecr get-login --profile artem --region eu-west-1 --no-include-email)" &> /dev/null'
                        echo "Debug: Building  with docker.build(${IMAGE_REPO}:${PROJECT_NAME}-${JOB_BASE_NAME}-${BUILD_NUMBER})"
                        sh "cp ./.jenkins/Dockerfile ./Dockerfile"
                        def image = docker.build ("${IMAGE_REPO}:${PROJECT_NAME}-${JOB_BASE_NAME}-${BUILD_NUMBER}")
                        echo "Debug: Before push to registry"
                        image.push ()
                        echo "Debug: After push to registry"
                    }
                }
            }
        }
        stage ('Deploy on staging.') {
            when { branch 'staging' }
            steps {
                sh "${KUBECTL} -n staging set image deployment.v1.apps/${PROJECT_NAME}-app ${PROJECT_NAME}=${IMAGE_REPO}:${PROJECT_NAME}-${JOB_BASE_NAME}-${BUILD_NUMBER} --record"
            }
        }
    }
    post {
        success {
            slackSend channel: "${SLACK_CHANNEL}", color: 'good', message: "Job: ${JOB_NAME}${BUILD_NUMBER} build was successful."
        }
        failure {
            slackSend channel: "${SLACK_CHANNEL}", color: 'danger', message: "Job: ${JOB_NAME}${BUILD_NUMBER} was finished with some error. It may occurs because of the build was rollbacked by docker swarm, or because of other error (watch the Jenkins Console Output): ${JOB_URL}${BUILD_ID}/consoleFull"
        }
        unstable {
            slackSend channel: "${SLACK_CHANNEL}", color: 'warning', message: "Job: ${JOB_NAME}${BUILD_NUMBER} was finished with some error. Please watch the Jenkins Console Output: ${JOB_URL}${BUILD_ID}/console."
        }
    }
}