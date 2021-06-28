// snykVersion=''
pipeline {
    agent { label 'jenkins_slave' }

    // Install the Jenkins tools you need for your project / environment
    tools {
        maven 'Maven-3.8.1' // Refers to a global tool configuration for Maven called 'Maven-3.8.1'
    }

    environment {
        SNYK_TOKEN = credentials('SNYK_ID_TOKEN') //Refers to SNYK token generated from portal
    }
    
    stages {

        stage('Initialize & Cleanup Workspace') {
            steps {
               echo 'Initialize & Cleanup Workspace'
               sh 'ls -la'
               sh 'rm -rf *'
               sh 'rm -rf .git'
               sh 'rm -rf .gitignore'
               sh 'ls -la'
            }
        }

        stage('Git Clone') {
            steps {
                git url: 'https://github.com/SaravananPerumal23/springboot-starter-app.git', branch: 'main'
                sh 'ls -la'
            }
        }

        stage('Test Build Requirements') {
            steps {
                sh 'java -version'
                sh 'mvn -v'
                // sh 'snykVersion=$(./snyk -v) echo snykVersion'
            }
        }

        // Not required if just install the Snyk CLI on your Agent
        stage('Download Snyk CLI') {
            steps {
                sh '''
                    latest_version=$(curl -Is "https://github.com/snyk/snyk/releases/latest"| grep "location:" | sed 's#.*tag/##g' | tr -d "\r")
                    echo "Latest Snyk CLI Version: ${latest_version}"

                    snyk_cli_dl_linux="https://github.com/snyk/snyk/releases/download/${latest_version}/snyk-linux"
                    echo "Download URL: ${snyk_cli_dl_linux}"

                    curl -Lo ./snyk "${snyk_cli_dl_linux}"
                    chmod +x ./snyk
                    ls -la
                    ./snyk -v
                '''
            }
        }

        stage('Build') {
            steps {
              sh 'mvn package -DskipTests=true'
            }
        }

        // Run snyk test to check for vulnerabilities and fail the build if any are found
        // Consider using --severity-threshold=<low|medium|high> for more granularity (see snyk help for more info).
        stage('Snyk Test using Snyk CLI') {
            steps {
                sh './snyk test --org=saravananperumal23 --project-name=snyk-test-app'
            }
        }

        // Capture the dependency tree for ongoing monitoring in Snyk.
        // This is typically done after deployment to some environment (ex staging, test, production, etc).
        stage('Snyk Monitor using Snyk CLI') {
            steps {
                // Use your own Snyk Organization with --org=<your-org>
                sh './snyk monitor --org=saravananperumal23 --project-name=snyk-test-app'
            }
        }
    }
}
