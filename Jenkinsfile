@Library('controlupdevops') _

pipeline {
    agent {
        label 'docker_servers'
    }
    options {
        sendSplunkConsoleLog()
    }
    environment {
        GIT_CODE_URL = "https://github.com/zivmitrani/python_flask.git"
        ACCOUNT = "zivmit"
        DOCKERHUB_REPO = "py-docker-images"
        DOCKER_IMAGE_NAME = "py_flask"
        IMAGE_VERSION = "latest"
    }
    parameters {
        choice(name: 'branch', choices: ['master', 'develop'], description: 'Source Branch')
     }
    stages {
        stage('Git Clone') {
            steps {
                script {
                    scmInfo = checkout([$class: 'GitSCM', branches: [[name: "*/${params.branch}"]], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'CleanBeforeCheckout']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'github_ziv', url: "${env.GIT_CODE_URL}"]]])
                    env.GIT_CODE_BRANCH = scmInfo.GIT_BRANCH
                    env.GIT_CODE_COMMIT = scmInfo.GIT_COMMIT
                    sh 'printenv'
                }
            }
        }
        stage('Build Preperations'){
            steps {
                script {
                    currentBuild.displayName = "${env.BUILD_VERSION}-#${env.BUILD_NUMBER}-(${params.branch})"
                    sh 'printenv'
                }
            }
        }
        stage("Build") {
            steps {
                sh (
                    script: "docker build -t ${env.ARTIFACTORY_SERVER_URL}/${env.DOCKERHUB_REPO}/${env.DOCKER_IMAGE_NAME}:${env.IMAGE_VERSION} -f services/app/Dockerfile .",
                    label: "Docker: build python_flask image "
                )
            }
        }
        stage("Publish to DockerHub"){
            steps {

                }
            }
        }

        stage("Deployment"){
            steps {
                sh (
                    script: "docker pull -t ${env.ACCOUNT}/${env.DOCKERHUB_REPO}/${env.DOCKER_IMAGE_NAME}:${env.BUILD_VERSION} -f services/app/Dockerfile .",
                    label: "Docker: pull python_flask image "
                )
                }
            }
        }

    }
    	cleanup {
    		cleanWs disableDeferredWipeout: false, deleteDirs: false
    	}
    }
}