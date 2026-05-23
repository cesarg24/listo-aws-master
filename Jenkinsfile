pipeline {
    agent any // Correcto para el Reto 1; se distribuirá en agentes en el Reto 3 [2, 3]

    environment {
        STACK_NAME      = 'staging-listo-aws-master'
        REGION          = 'us-east-1'
        S3_BUCKET       = 'pruebacesarg'
        REPO_URL        = 'https://github.com/cesarg24/listo-aws-master.git'
        GIT_CREDENTIALS = 'github-credentials-id'
    }

    stages {
        // 1. Etapa de obtención de código
        stage('Get Code') {
            steps {
                git branch: 'develop',
                    credentialsId: env.GIT_CREDENTIALS,
                    url: env.REPO_URL
            }
        }

        // 2. Etapa de Pruebas Estáticas [1]
        stage('Static Test') {
            steps {
                echo "Paso 2: Aquí ejecutaremos Flake8 y Bandit para analizar el código en /src y publicaremos los informes."
            }
        }

        // 3. Etapa de Despliegue con SAM [4]
        stage('Deploy') {
            steps {
                echo "Paso 3: Aquí utilizaremos comandos de AWS SAM para construir y desplegar en Staging de forma automática."
            }
        }

        // 4. Etapa de Pruebas Rest (Integración) [5]
        stage('Rest Test') {
            steps {
                echo "Paso 4: Aquí ejecutaremos Pytest o comandos Curl para validar la API. Si falla, el pipeline se detendrá."
            }
        }

        // 5. Etapa de Promoción [6]
        stage('Promote') {
            steps {
                echo "Paso 5: Si todo ha sido exitoso, realizaremos el merge de la rama 'develop' a 'master' usando git."
            }
        }
    }
}
