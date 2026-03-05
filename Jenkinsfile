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
        stage('Static Test') {
          steps {
            unstash 'codigo'
            sh '''
              rm -f flake8.out bandit.out
              python3 -m flake8 --exit-zero --format=pylint src > flake8.out || true
              python3 -m bandit -r src -f custom -o bandit.out -ll \
                --msg-template "{abspath}:{line}: [{test_id}] {msg}" || true
              touch bandit.out
            '''
            recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')]
            recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')]
          }
        }
        stage('Deploy') {
          steps {
            unstash 'codigo'
            sh '''
              set -e
        
              sam build
              sam validate
        
              sam deploy \
                --config-env default \
                --region us-east-1 \
                --no-confirm-changeset \
                --no-fail-on-empty-changeset
        
              BASE_URL=$(aws cloudformation describe-stacks \
                --stack-name staging-todolist-aws \
                --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' \
                --output text \
                --region us-east-1)
        
              echo "export BASE_URL=${BASE_URL}" > env.sh
              chmod +x env.sh
              echo "La URL de la API es: ${BASE_URL}"
            '''
            stash name: 'env', includes: 'env.sh'
          }
        }
        stage('Rest Test') {
          steps {
            unstash 'codigo'
            unstash 'env'
            sh '''
              set -e
              . ./env.sh
              echo "Probando contra: $BASE_URL"
        
              python3 -m pytest --junit-xml=result-int.xml test/integration/todoApiTest.py
            '''
            junit allowEmptyResults: true, testResults: 'result-int.xml'
          }
        }
        stage('Promote') {
          steps {
            unstash 'codigo'
            sh '''
              set -e
              git fetch origin
              git checkout -B master origin/master
              git config merge.ours.driver true
              git merge origin/develop --no-edit
              git push origin master
            '''
          }
        }        
      }
    }
        