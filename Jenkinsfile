// Jenkinsfile - Pipeline de "Sistema Alumnos" (OCY1102) - Guía 3.4.2 Documentación y trazabilidad
pipeline {
    agent any

    options {
        timestamps()
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    environment {
        PIPENV_VENV_IN_PROJECT    = '1'
        PIPENV_IGNORE_VIRTUALENVS = '1'
        PIPENV_NOSPIN             = '1'
    }

    stages {
        stage('Build') {
            steps {
                script {
                    def sha = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
                    env.GIT_SHA     = sha
                    env.APP_VERSION = "1.0.${env.BUILD_NUMBER}+${sha}"
                }
                echo "Versión de este build: ${env.APP_VERSION}"
                sh 'rm -rf .venv reports && mkdir -p reports'
                sh 'pipenv install --dev --deploy'
                sh 'pipenv run python -m compileall -q app.py'
            }
        }

        stage('Test') {
            steps {
                sh 'pipenv run pytest --junitxml=reports/junit.xml'
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'reports/junit.xml'
                }
            }
        }

        stage('Generate Documentation') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
                    sh '''#!/bin/bash
                        set -eu
                        mkdir -p reports
                        rm -rf docs/html reports/doxygen_warnings.log
                        doxygen Doxyfile
                        if [ -s reports/doxygen_warnings.log ]; then
                            echo "Doxygen encontró código sin documentar o mal documentado:"
                            cat reports/doxygen_warnings.log
                            exit 1
                        fi
                        echo "Documentación generada sin advertencias en docs/html"
                    '''
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'docs/html/**', allowEmptyArchive: true
                }
            }
        }

        // >>> [B-VERSION] aquí irá la etapa Version Control
        // >>> [B-DEPLOY] aquí irá la etapa Deploy
    }

    post {
        always {
            archiveArtifacts artifacts: 'reports/**', allowEmptyArchive: true
        }
    }
}