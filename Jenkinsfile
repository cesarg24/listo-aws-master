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
