pipeline {
    agent none
    stages {
        stage('Semgrep Builder') {
            agent {
                dockerfile {
                    filename 'Dockerfile'
                    dir 'semgrep'
                    additionalBuildArgs '--tag devsecops/semgrep --network host'
                } }
            steps {
                sh 'echo "SEMGREP INSTALLED"'
            }
            }
        stage('SAST') {
            agent {
                docker {
                    image 'devsecops/semgrep'
                    args '-u root --privileged --network host'
            }
        }
            steps {
                sh 'semgrep --version'
                sh "semgrep --config p/expressjs --junit-xml -o ${env.WORKSPACE}/semgrep.junit.xml"
                archiveArtifacts 'semgrep.junit.xml'
                xunit(
                    tools: [GoogleTest(pattern : 'semgrep.junit.xml')], 
                    thresholds: [failed(failureThreshold: '100')],
                    )
            }
        }
        stage('OWASP Dependency-Check Vulnerabilities') {
        agent any
      steps {
        dependencyCheck additionalArguments: ''' 
                    -o './'
                    -s './'
                    -f 'ALL' 
                    --prettyPrint''', odcInstallation: 'OWASP Dependency-Check Vulnerabilities'
        
        dependencyCheckPublisher pattern: 'dependency-check-report.xml'
      }
    }
    }
}

