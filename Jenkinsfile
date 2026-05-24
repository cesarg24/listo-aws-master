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
                    echo "=== Ejecutando Flake8 ==="
                    flake8 src/ --format=pylint --output-file=flake8.out || true
        
                    echo "=== Ejecutando Bandit ==="
                    bandit -r src/ --msg-template "{abspath}:{line}: [{severity}] {test_id}: {msg}" -f custom -o bandit.out || true
                '''
                recordIssues(
                    tools: [
                        pyLint(name: 'Flake8 Style', pattern: 'flake8.out'),
                        pyLint(name: 'Bandit Security', pattern: 'bandit.out')
                    ]
                )
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
