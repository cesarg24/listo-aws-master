pipeline {
    agent any // Correcto para el Reto 1; se distribuirá en agentes en el Reto 3

    environment {
        STACK_NAME      = 'staging-listo-aws-master'
        REGION          = 'us-east-1'
        //S3_BUCKET       = 'pruebacesarg'
        REPO_URL        = 'https://github.com/cesarg24/todo-list-aws.git'
        GIT_CREDENTIALS = 'github-credentials-id'
    }

    stages {
        // Primera Etapa de obtención de código
        stage('Get Code') {
            steps {
                git branch: 'develop',
                    credentialsId: env.GIT_CREDENTIALS,
                    url: env.REPO_URL
                              
                sh '''
                    echo "=== Descargando configuración de Staging ==="
                    wget https://raw.githubusercontent.com/cesarg24/todo-list-aws-config/staging/samconfig.toml \
                    -O samconfig.toml
                    echo "=== Contenido del samconfig.toml ==="
                    cat samconfig.toml
                '''
	    }
        }

        // Segunda Etapa de Pruebas Estáticas
      stage('Static Test') {
            steps {
                sh '''
                    echo "=== Ejecutando Flake8 ==="
                    # Usamos el formato estándar para que el plugin nativo lo reconozca
                    flake8 src/ --output-file=flake8-report.txt || true

                    echo "=== Ejecutando Bandit ==="
                    # Configuración para Bandit
                    bandit -r src/ --msg-template "{abspath}:{line}: [{severity}] {test_id}: {msg}" -f custom -o bandit-report.out || true
                '''

                // Publicamos ambos con sus herramientas específicas para que salgan en el menú
                recordIssues(
                    tools: [
                        flake8(name: 'Flake8 Style', pattern: 'flake8-report.txt'),
                        pyLint(name: 'Bandit Security', pattern: 'bandit-report.out')
                    ]
                )
            }
        }


        // Tercera. Etapa de Despliegue con SAM
       stage('Deploy') {
            steps {
                sh '''
                    echo "=== SAM Build ==="
                    sam build
        
                    echo "=== SAM Validate ==="
                    sam validate --region us-east-1
        
                    echo "=== SAM Deploy Staging ==="
                    sam deploy --config-env staging --resolve-s3
                '''
            }
        }

        // Cuarta. Etapa de Pruebas Rest (Integración)
     stage('Rest Test') {
            steps {
                sh '''
                    echo "=== Obteniendo la URL de la API ==="
                    API_URL=$(aws cloudformation describe-stacks \
                        --stack-name todo-list-aws-staging \
                        --region us-east-1 \
                        --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' \
                        --output text)
        
                    echo "API URL: ${API_URL}"
        
                    echo "=== Ejecutando las Pruebas de Integración ==="
                    BASE_URL=${API_URL} pytest test/integration/todoApiTest.py -v --tb=short
                '''
            }
        }

        // Quinta Etapa de Promoción
     stage('Promote') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: env.GIT_CREDENTIALS,
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_PASS'
                )]) {
                    sh '''
                        echo "=== Merge develop → main probando==="
                        git config user.email "jenkins@ci.local"
                        git config user.name "Jenkins CI"
                        
                        git stash        
                        git fetch origin main
                        git checkout -b main origin/main
                        git merge develop -X ours
                        git push https://${GIT_USER}:${GIT_PASS}@github.com/cesarg24/todo-list-aws.git main
                    '''
                }
            }
        }
    }
}
