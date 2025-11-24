// example Jenkinsfile for Coverity scans using the Black Duck Security Scan Plugin

// https://plugins.jenkins.io/blackduck-security-scan

pipeline {
    agent any
    environment {
        REPO_NAME = "${env.GIT_URL.tokenize('/.')[-2]}"
    }
    tools {
        maven 'maven-3.9'
        jdk 'openjdk-17'
    }
    stages {
        stage('Detect Branch') {
            steps {
                script {
                    // Try multiple methods to get branch name
                    def branchName = env.BRANCH_NAME
                    if (!branchName || branchName == 'null') {
                        branchName = env.GIT_BRANCH ?: sh(
                            script: 'git rev-parse --abbrev-ref HEAD',
                            returnStdout: true
                        ).trim()
                    }
                    // Remove origin/ prefix if present
                    branchName = branchName.replaceAll('^origin/', '')
                    env.BRANCH_NAME = branchName
                    
                    // Set FULLSCAN and PRSCAN flags
                    def fullScanBranches = ['main', 'master', 'develop', 'stage', 'release']
                    env.FULLSCAN = fullScanBranches.contains(branchName) ? 'true' : 'false'
                    
                    def prTarget = env.CHANGE_TARGET ?: ''
                    env.PRSCAN = fullScanBranches.contains(prTarget) ? 'true' : 'false'
                    
                    echo "Branch detected: ${branchName}"
                    echo "FULLSCAN: ${env.FULLSCAN}"
                    echo "PRSCAN: ${env.PRSCAN}"
                }
            }
        }
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
