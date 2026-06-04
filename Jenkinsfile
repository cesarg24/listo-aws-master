pipeline {
    agent any 

    environment {
        STACK_NAME      = 'todo-list-aws-staging' 
        REGION          = 'us-east-1'
        REPO_URL        = 'https://github.com/cesarg24/todo-list-aws.git'
        GIT_CREDENTIALS = 'github-credentials-id'
    }

    stages {
        // Reto 4: Obtención de código y configuración externa
        stage('Get Code') {
            steps {
                git branch: 'develop', credentialsId: env.GIT_CREDENTIALS, url: env.REPO_URL
                sh '''
                    echo "=== Descargando configuración de Staging (Reto 4) ==="
                    wget https://raw.githubusercontent.com/cesarg24/todo-list-aws-config/staging/samconfig.toml -O samconfig.toml
                    cat samconfig.toml
                '''
            }
        }

        // Reto 1: Análisis Estático (Flake8 y Bandit)
        stage('Static Test') {
            steps {
                sh '''
                    flake8 src/ --output-file=flake8-report.txt || true
                    bandit -r src/ --msg-template "{abspath}:{line}: [{severity}] {test_id}: {msg}" -f custom -o bandit-report.out || true
                '''
                recordIssues(tools: [flake8(name: 'Flake8 Style', pattern: 'flake8-report.txt'), pyLint(name: 'Bandit Security', pattern: 'bandit-report.out')])
            }
        }

        // Calidad Técnica: Tests Unitarios y Cobertura
        stage('Unit Test & Coverage') {
            steps {
                sh '''
		     echo "=== Ejecutando Tests y Cobertura con herramienta global ==="
                    # Usamos python3-coverage que es el comando estándar en Ubuntu
                    python3-coverage run --branch --source=src -m pytest test/unit/ --junitxml=result-unit.xml
                    python3-coverage xml -o coverage.xml
                    python3-coverage report 
                '''
                post{
		    always{
                      junit 'result-unit.xml'
                   }
                 }  
	     }
        }

        // Reto 1: Despliegue en Staging
        stage('Deploy Staging') {
            steps {
                sh '''
                    sam build
                    sam deploy --config-env staging --resolve-s3 --no-confirm-changeset --no-fail-on-empty-changeset
                '''
            }
        }

        // Reto 1: Pruebas de Integración (Rest Test)
        stage('Rest Test') {
            steps {
                sh '''
                    API_URL=$(aws cloudformation describe-stacks --stack-name ${STACK_NAME} --region ${REGION} --query 'Stacks.Outputs[?OutputKey==`BaseUrlApi`].OutputValue' --output text)
                    BASE_URL=${API_URL} pytest test/integration/todoApiTest.py -v --tb=short
                '''
            }
        }

        // Reto 1: Promoción (Merge automático a Main)
        stage('Promote') {
            steps {
                withCredentials([usernamePassword(credentialsId: env.GIT_CREDENTIALS, usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                    sh '''
                        git config user.email "jenkins@ci.local"
                        git config user.name "Jenkins CI"
                        git fetch origin main
                        git checkout main || git checkout -b main origin/main
                        git merge develop -X ours
                        git push https://${GIT_USER}:${GIT_PASS}@github.com/cesarg24/todo-list-aws.git main
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "=== Limpiando Workspace (Requerimiento Reto 1) ==="
            cleanWs() // Obligatorio según la guía para higiene de agentes
        }
    }
}
