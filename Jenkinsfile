#!groovy

@Library('github.com/red-panda-ci/jenkins-pipeline-library@v3.1.6') _

// Initialize global config
cfg = jplConfig('docker-flutter-builder', 'docker', '', [slack: '', email:'pedroamador.rodriguez+teecke@gmail.com'])

pipeline {
    agent { label 'docker' }

    stages {
        stage ('Initialize') {
            steps  {
                jplStart(cfg)
            }
        }
        stage ('Bash linter') {
            steps {
                script {
                    sh "devcontrol run-bash-linter"
                }
            }
        }
        stage ('Build') {
            steps {
                script {
                    sh "devcontrol build-all"
                }
            }
        }
        stage('Make release') {
            when { expression { cfg.BRANCH_NAME.startsWith('release/new') } }
            steps {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'teeckebot-docker-credentials',
                                usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
                    sh '''docker login -u ${USERNAME} -p ${PASSWORD}
                    docker push teecke/docker-flutter-builder
                    '''
                }
                jplMakeRelease(cfg, true)
            }
            post {
                always {
                    sh 'docker logout'
                }
            }
        }
    }

    post {
        always {
            jplPostBuild(cfg)
        }
    }

    options {
        timestamps()
        ansiColor('xterm')
        buildDiscarder(logRotator(artifactNumToKeepStr: '20',artifactDaysToKeepStr: '30'))
        disableConcurrentBuilds()
        timeout(time: 180, unit: 'MINUTES')
    }
}
