pipeline {
    agent any

    parameters {
        string(name: 'REPO_URL', defaultValue: 'https://github.com/ТВОЙ_НИК/cicd-demo-app.git', description: 'HTTPS URL of the repository')
        string(name: 'GIT_BRANCH', defaultValue: 'master', description: 'Branch to build')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run unit tests')
    }

    stages {
        stage('Checkout') {
            steps {
                deleteDir()
                git url: params.REPO_URL, branch: params.GIT_BRANCH
                script {
                    env.GIT_SHA = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                }
                echo "Branch: ${params.GIT_BRANCH}"
                echo "Commit SHA: ${env.GIT_SHA}"
            }
        }

        stage('Validate') {
            steps {
                sh '''
                    for f in README.md requirements.txt; do
                        [ -f "$f" ] || { echo "ERROR: file $f not found"; exit 1; }
                    done
                    for d in src tests; do
                        [ -d "$d" ] || { echo "ERROR: directory $d not found"; exit 1; }
                    done
                    echo "Project structure is OK"
                '''
            }
        }

        stage('Test') {
            when { expression { return params.RUN_TESTS } }
            steps {
                sh 'python3 -m unittest discover -s tests -v'
            }
        }

        stage('Build') {
            steps {
                sh '''
                    echo "Build number: ${BUILD_NUMBER}" > build-info.txt
                    echo "Commit SHA: ${GIT_SHA}" >> build-info.txt
                    tar -czf app.tar.gz src tests README.md requirements.txt build-info.txt
                '''
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'app.tar.gz', fingerprint: true
            }
        }
    }

    post {
        success { echo 'Build finished SUCCESSFULLY' }
        failure { echo 'Build FAILED' }
    }
}
EOF
