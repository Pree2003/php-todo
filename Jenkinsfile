pipeline {
    agent any

    environment {
        DB_HOST = '172.31.32.84'
        DB_DATABASE = 'homestead'
    }

    stages {

        stage("Initial cleanup") {
            steps {
                dir("${WORKSPACE}") {
                    deleteDir()
                }
            }
        }

        stage('Checkout SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/Pree2003/php-todo.git'
            }
        }

        stage('Prepare Environment') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'homestead-db',
                        usernameVariable: 'DB_USERNAME',
                        passwordVariable: 'DB_PASSWORD'
                    )
                ]) {
                    sh '''
                        cat > .env <<EOF
DB_CONNECTION=mysql
DB_HOST=${DB_HOST}
DB_PORT=3306
DB_DATABASE=${DB_DATABASE}
DB_USERNAME=${DB_USERNAME}
DB_PASSWORD=${DB_PASSWORD}
EOF

                        mkdir -p bootstrap/cache
                    '''
                }
            }
        }

        stage('Prepare Dependencies') {
            steps {
                sh '''
                    docker run --rm \
                    -u "$(id -u):$(id -g)" \
                    -v "$WORKSPACE":/app \
                    -w /app \
                    php-todo-legacy:latest \
                    composer install
                '''
            }
        }

        stage('Database Migration') {
            steps {
                sh '''
                    docker run --rm \
                    -u "$(id -u):$(id -g)" \
                    -v "$WORKSPACE":/app \
                    -w /app \
                    php-todo-legacy:latest \
                    php artisan migrate --force
                '''
            }
        }

        stage('Database Seed') {
            steps {
                sh '''
                    docker run --rm \
                    -u "$(id -u):$(id -g)" \
                    -v "$WORKSPACE":/app \
                    -w /app \
                    php-todo-legacy:latest \
                    php artisan db:seed --force
                '''
            }
        }

                stage('Generate Application Key') {
            steps {
                sh '''
                    docker run --rm \
                    -u "$(id -u):$(id -g)" \
                    -v "$WORKSPACE":/app \
                    -w /app \
                    php-todo-legacy:latest \
                    php artisan key:generate
                '''
            }
        }
    }

    post {
        always {
            sh 'rm -f .env'
        }
    }
}
                      
