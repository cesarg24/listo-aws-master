pipeline {
    agent any // Correcto para el Reto 1; se distribuirá en agentes en el Reto 3

    environment {
        STACK_NAME      = 'staging-listo-aws-master'
        REGION          = 'us-east-1'
        S3_BUCKET       = 'pruebacesarg'
        REPO_URL        = 'https://github.com/cesarg24/listo-aws-master.git'
        GIT_CREDENTIALS = 'github-credentials-id'
    }

    stages {
        // Primera Etapa de obtención de código
        stage('Get Code') {
            steps {
                git branch: 'develop',
                    credentialsId: env.GIT_CREDENTIALS,
                    url: env.REPO_URL
            }
        }

        // Segunda Etapa de Pruebas Estáticas
        stage('Static Test') {
            steps {
                sh '''
                    echo "=== Flake8 ==="
                    flake8 src/ --output-file=flake8-report.txt || true
        
                    echo "=== Bandit ==="
                    bandit -r src/ -f json -o bandit-report.json || true
                '''
            }
            post {
                always {
                    recordIssues(
                        tools: [
                            flake8(pattern: 'flake8-report.txt'),
                            pyLint(pattern: 'bandit-report.json')
                        ]
                    )
                    archiveArtifacts artifacts: 'flake8-report.txt, bandit-report.json',
                                     allowEmptyArchive: true
                }
            }
        } 

        // 3. Etapa de Despliegue con SAM
        stage('Deploy') {
            steps {
                echo "Paso 3: Aquí utilizaremos comandos de AWS SAM para construir y desplegar en Staging de forma automática."
            }
        }

        // 4. Etapa de Pruebas Rest (Integración)
        stage('Rest Test') {
            steps {
                echo "Paso 4: Aquí ejecutaremos Pytest o comandos Curl para validar la API. Si falla, el pipeline se detendrá."
            }
        }

        // 5. Etapa de Promoción
        stage('Promote') {
            steps {
                echo "Paso 5: Si todo ha sido exitoso, realizaremos el merge de la rama 'develop' a 'master' usando git."
            }
        }
    }
}
