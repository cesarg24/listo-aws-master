pipeline {
    agent any

    environment {
        STACK_NAME      = 'todo-list-aws-production'
        REGION          = 'us-east-1'
        REPO_URL        = 'https://github.com/cesarg24/todo-list-aws.git'
        DYNAMODB_TABLE = 'todo-list-aws-staging'
        GIT_CREDENTIALS = 'github-credentials-id'
    }

    stages {

        stage('Get Code') {
            steps {
                git branch: 'main',
                    credentialsId: env.GIT_CREDENTIALS,
                    url: env.REPO_URL
		sh '''
                    echo "=== Descargando configuración de Production ==="
                    wget https://raw.githubusercontent.com/cesarg24/todo-list-aws-config/production/samconfig.toml \
                    -O samconfig.toml
                    echo "=== Contenido del samconfig.toml ==="
                    cat samconfig.toml
                 '''






            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    echo "=== SAM Build ==="
                    sam build

                    echo "=== SAM Validate ==="
                    sam validate --region us-east-1

                    echo "=== SAM Deploy Production ==="
                    sam deploy --config-env production --resolve-s3
                '''
            }
        }

        stage('Rest Test') {
            steps {
                sh '''
                    echo "=== Obteniendo la URL de la API ==="
                    API_URL=$(aws cloudformation describe-stacks \
                        --stack-name todo-list-aws-production \
                        --region us-east-1 \
                        --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' \
                        --output text)

                    echo "API URL: ${API_URL}"

                    echo "=== Ejecutando Pruebas de Integración ==="
                    BASE_URL=${API_URL} pytest test/integration/todoApiTest.py -v --tb=short
                '''
            }
        }

    }
}
