pipeline{
    agent any

    options {
        skipDefaultCheckout(true)
        }  
    stages {       
        stage('Get Code')
        {
            steps {
                checkout scm
                sh 'ls -la'
                stash name:'codigo', includes:'**'
                }
        }
        stage('Static') {
        agent any
        steps {
            unstash 'codigo'
            sh '''
            flake8 --format=pylint --exit-zero src > flake8.out
            bandit -r src -f sarif -o bandit.sarif || true
            '''
            catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
            script {
                recordIssues(
                    tools: [
                        flake8(name: 'Flake8', pattern: 'flake8.out'),
                        sarif(pattern: 'bandit.sarif')
                    ]
                )
            }
            }
        }
        }
        }
        }
        