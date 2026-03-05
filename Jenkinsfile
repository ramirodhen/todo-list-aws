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
              python3 -m flake8 --exit-zero --format=pylint src > flake8.out
              python3 -m bandit -r src -f custom -o bandit.out -ll \
                --msg-template "{abspath}:{line}: [{test_id}] {msg}" || true
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
                --resolve-s3 \
                --stack-name "todo-list-aws-staging" \
                --region "us-east-1" \
                --capabilities CAPABILITY_IAM \
                --no-confirm-changeset \
                --no-fail-on-empty-changeset \
                --parameter-overrides Stage=staging
        
              BASE_URL=$(aws cloudformation describe-stacks \
                --stack-name todo-list-aws-staging \
                --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' \
                --output text)
        
              echo "export BASE_URL=${BASE_URL}" > env.sh
              chmod a+x env.sh
              echo "La URL de la API es: ${BASE_URL}"
            '''
            stash name: 'env', includes: 'env.sh'
          }
        }
        }
        }
        