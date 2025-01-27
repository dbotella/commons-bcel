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
        stage('') {
            steps {
                // Clean before build
                cleanWs()
                // We need to explicitly checkout from SCM here
                checkout scm
                sh 'mvn -Drat.skip=true -DskipTests=true package'
            }
        }
        stage('Coverity Full Scan') {
            when { not { changeRequest() } }
            steps {
                script {
                    def status = security_scan product: 'coverity',
                    coverity_url: "${COVERITY_URL}",
                    coverity_user: "${COVERITY_USER_NAME}",
                    coverity_passphrase: "${COVERITY_PASSWORD}",
                    coverity_stream_name: "${COVERITY_STREAM_NAME}",
                    coverity_project_name: "${COVERITY_PROJECT_NAME}",
                    coverity_install_directory: '/opt/cov-analysis-linux64-2024.12.0',
                    coverity_local: true,
                    coverity_build_command: 'mvn -Drat.skip=true -DskipTests=true package'
                    coverity_clean_command: 'mvn clean'

                    // Uncomment to add custom logic based on return status
                    if (status == 8) { unstable 'policy violation' }
                else if (status != 0) { error 'plugin failure' }
                }
            }
        }
        stage('Coverity PR Scan') {
            when { changeRequest() }
            steps {
                script {
                    def status = security_scan product: 'coverity',
                    coverity_url: "${COVERITY_URL}",
                    coverity_user: "${COVERITY_USER_NAME}",
                    coverity_passphrase: "${COVERITY_PASSWORD}",
                    coverity_stream_name: "${COVERITY_STREAM_NAME}",
                    coverity_project_name: "${COVERITY_PROJECT_NAME}",
                    coverity_install_directory: '/opt/cov-analysis-linux64-2024.12.0',
                    coverity_local: true,
                    coverity_automation_prcomment: true

                    // Uncomment to add custom logic based on return status
                    if (status == 8) { unstable 'policy violation' }
                else if (status != 0) { error 'plugin failure' }
                }
            }
        }
    }
    post {
        always {
            //zip archive: true, dir: '.bridge', zipFile: 'bridge-logs.zip'
            cleanWs(cleanWhenNotBuilt: false,
                    deleteDirs: true,
                    disableDeferredWipeout: true,
                    notFailBuild: true,
                    patterns: [[pattern: '.gitignore', type: 'INCLUDE'],
                               [pattern: '.propsfile', type: 'EXCLUDE']])
        
        }
    }
}
