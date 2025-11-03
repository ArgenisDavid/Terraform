pipeline {
    agent any
    
    environment {
        DOCKER_HOST = "unix:///var/run/docker.sock"
    }
    
    stages {
        stage('Verify Environment') {
            steps {
                sh '''
                    echo "=== Verificando herramientas ==="
                    docker --version
                    docker-compose --version
                    python --version
                    echo "=== Estructura del proyecto ==="
                    pwd
                    ls -la
                    echo "=== Contenido de app/ ==="
                    ls -la app/
                    echo "=== Contenido de db/ ==="  
                    ls -la db/
                '''
            }
        }
        
        stage('Build') {
            steps {
                sh 'docker-compose build --no-cache'
            }
        }
        
        stage('Test') {
            steps {
                sh 'docker-compose -f docker-compose.test.yml up --abort-on-container-exit --exit-code-from test-web'
            }
            post {
                always {
                    sh 'docker-compose -f docker-compose.test.yml down'
                }
            }
        }
        
        stage('Deploy to Dev') {
            steps {
                sh 'docker-compose down || true'
                sh 'docker-compose up -d'
                sh 'sleep 15'
            }
        }
        
        stage('Smoke Test') {
            steps {
                sh '''
                    echo "=== Realizando smoke test ==="
                    timeout time: 30, unit: 'SECONDS', activity: true {
                        while true; do
                            if curl -s http://localhost:5000/login > /dev/null; then
                                echo "✅ Aplicación Flask respondiendo correctamente"
                                break
                            else
                                echo "⏳ Esperando que la aplicación esté lista..."
                                sleep 5
                            fi
                        done
                    }
                '''
            }
        }
    }
    
    post {
        always {
            sh 'docker-compose down || true'
            cleanWs()
        }
        success {
            echo "🎉 Pipeline completado exitosamente!"
        }
        failure {
            echo "❌ Pipeline falló - Revisar logs"
        }
    }
}
