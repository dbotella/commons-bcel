pipeline {
    agent any
    tools {
        maven 'maven-3.9'
        jdk 'openjdk-17'
    }
    environment {
        REPO_NAME = "${env.GIT_URL.tokenize('/.')[-2]}"
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn -Drat.skip=true -DskipTests=true package'
            }
        }
        stage('Coverity Full Scan') {
            when { not { changeRequest() } }
            steps {
                sh 'chmod 400 auth-key.txt'
                synopsys_scan product: 'coverity', coverity_project_name: "${env.REPO_NAME}", coverity_stream_name: "${env.REPO_NAME}",
                    coverity_policy_view: 'Outstanding Issues', coverity_install_directory: '/opt/cov-analysis-linux64-2023.12.0'
            }
        }
        stage('Coverity PR Scan') {
            when { changeRequest() }
            steps {
                sh 'chmod 400 auth-key.txt'
                synopsys_scan product: 'coverity', coverity_project_name: "${env.REPO_NAME}", coverity_stream_name: "${env.REPO_NAME}",
                    coverity_automation_prcomment: true, coverity_install_directory: '/opt/cov-analysis-linux64-2023.12.0'
            }
        }
    }
    post {
        always {
            //zip archive: true, dir: '.bridge', zipFile: 'bridge-logs.zip'
            cleanWs()
        }
    }
}