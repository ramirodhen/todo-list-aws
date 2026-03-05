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
      }
    }
        