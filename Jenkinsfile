// example Jenkinsfile for Coverity scans using the Black Duck Security Scan Plugin

// https://plugins.jenkins.io/blackduck-security-scan

pipeline {
    agent any
    environment {
        REPO_NAME = "${env.GIT_URL.tokenize('/.')[-2]}"
        FULLSCAN = "${env.BRANCH_NAME ==~ /^(main|master|develop|stage|release)$/ ? 'true' : 'false'}"
        PRSCAN = "${env.CHANGE_TARGET ==~ /^(main|master|develop|stage|release)$/ ? 'true' : 'false'}"
    }
    tools {
        maven 'maven-3.9'
        jdk 'openjdk-17'
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn -B -Drat.skip=true -DskipTests=true package'
            }
        }
        stage('Coverity') {
            when {
                anyOf {
                    environment name: 'FULLSCAN', value: 'true'
                    environment name: 'PRSCAN', value: 'true'
                }
            }
            steps {
                security_scan product: 'coverity',
                    coverity_project_name: "$REPO_NAME",
                    coverity_stream_name: "$REPO_NAME-$BRANCH_NAME",
                    coverity_args: "-o commit.connect.description=$BUILD_TAG",
                    coverity_policy_view: '(Global) Newly Introduced Defects',
                    coverity_prComment_enabled: true,
                    coverity_local: true,
                    coverity_install_directory: '/opt/cov-analysis-linux64-2025.9.0',
                    coverity_build_command: 'mvn -B -Drat.skip=true -DskipTests package',
                    coverity_clean_command: 'mvn -B clean',
                    include_diagnostics: false,
                    mark_build_status: 'UNSTABLE'
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
