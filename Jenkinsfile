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



        stage('Code Analysis') {
            steps {
                sh '''
                    mkdir -p build/logs

                    docker run --rm \
                    -u "$(id -u):$(id -g)" \
                    -e COMPOSER_HOME=/tmp/composer \
                    -v "$WORKSPACE":/app \
                    -w /app \
                    php-todo-legacy:latest \
                    sh -c 'composer global require phploc/phploc:4.0.1 --no-interaction --prefer-dist && "$COMPOSER_HOME/vendor/bin/phploc" app/ --log-csv build/logs/phploc.csv'
                '''
            }
        }


        stage('Plot Code Analysis Report') {
            steps {
                plot csvFileName: 'phploc-lines.csv',
                    csvSeries: [[
                        file: 'build/logs/phploc.csv',
                        inclusionFlag: 'INCLUDE_BY_STRING',
                        exclusionValues: 'Lines of Code (LOC),Comment Lines of Code (CLOC),Non-Comment Lines of Code (NCLOC),Logical Lines of Code (LLOC)',
                        displayTableFlag: false,
                        url: ''
                    ]],
                    group: 'phploc', numBuilds: '100',
                    style: 'line', title: 'A - Lines of Code',
                    yaxis: 'Lines of Code'

                plot csvFileName: 'phploc-containers.csv',
                    csvSeries: [[
                        file: 'build/logs/phploc.csv',
                        inclusionFlag: 'INCLUDE_BY_STRING',
                        exclusionValues: 'Directories,Files,Namespaces',
                        displayTableFlag: false,
                        url: ''
                    ]],
                    group: 'phploc', numBuilds: '100',
                    style: 'line', title: 'B - Structure Containers',
                    yaxis: 'Count'

                plot csvFileName: 'phploc-length.csv',
                    csvSeries: [[
                        file: 'build/logs/phploc.csv',
                        inclusionFlag: 'INCLUDE_BY_STRING',
                        exclusionValues: 'Classes Length (LLOC),Functions Length (LLOC),Average Function Length (LLOC)',
                        displayTableFlag: false,
                        url: ''
                    ]],
                    group: 'phploc', numBuilds: '100',
                    style: 'line', title: 'C - Code Structure Length',
                    yaxis: 'Logical Lines of Code'

                plot csvFileName: 'phploc-complexity.csv',
                    csvSeries: [[
                        file: 'build/logs/phploc.csv',
                        inclusionFlag: 'INCLUDE_BY_STRING',
                        exclusionValues: 'Cyclomatic Complexity / Lines of Code,Cyclomatic Complexity / Number of Methods',
                        displayTableFlag: false,
                        url: ''
                    ]],
                    group: 'phploc', numBuilds: '100',
                    style: 'line', title: 'D - Relative Cyclomatic Complexity',
                    yaxis: 'Complexity'

                plot csvFileName: 'phploc-classes.csv',
                    csvSeries: [[
                        file: 'build/logs/phploc.csv',
                        inclusionFlag: 'INCLUDE_BY_STRING',
                        exclusionValues: 'Classes,Abstract Classes,Concrete Classes',
                        displayTableFlag: false,
                        url: ''
                    ]],
                    group: 'phploc', numBuilds: '100',
                    style: 'line', title: 'E - Types of Classes',
                    yaxis: 'Count'

                plot csvFileName: 'phploc-methods.csv',
                    csvSeries: [[
                        file: 'build/logs/phploc.csv',
                        inclusionFlag: 'INCLUDE_BY_STRING',
                        exclusionValues: 'Methods,Non-Static Methods,Static Methods,Public Methods,Non-Public Methods',
                        displayTableFlag: false,
                        url: ''
                    ]],
                    group: 'phploc', numBuilds: '100',
                    style: 'line', title: 'F - Types of Methods',
                    yaxis: 'Count'

                plot csvFileName: 'phploc-constants.csv',
                    csvSeries: [[
                        file: 'build/logs/phploc.csv',
                        inclusionFlag: 'INCLUDE_BY_STRING',
                        exclusionValues: 'Constants,Global Constants,Class Constants',
                        displayTableFlag: false,
                        url: ''
                    ]],
                    group: 'phploc', numBuilds: '100',
                    style: 'line', title: 'G - Types of Constants',
                    yaxis: 'Count'

                plot csvFileName: 'phploc-testing.csv',
                    csvSeries: [[
                        file: 'build/logs/phploc.csv',
                        inclusionFlag: 'INCLUDE_BY_STRING',
                        exclusionValues: 'Test Classes,Test Methods',
                        displayTableFlag: false,
                        url: ''
                    ]],
                    group: 'phploc', numBuilds: '100',
                    style: 'line', title: 'H - Testing',
                    yaxis: 'Count'

                plot csvFileName: 'phploc-logical-lines.csv',
                    csvSeries: [[
                        file: 'build/logs/phploc.csv',
                        inclusionFlag: 'INCLUDE_BY_STRING',
                        exclusionValues: 'Logical Lines of Code (LLOC),Classes Length (LLOC),Functions Length (LLOC),LLOC outside functions or classes',
                        displayTableFlag: false,
                        url: ''
                    ]],
                    group: 'phploc', numBuilds: '100',
                    style: 'line', title: 'I - Logical Lines by Structure',
                    yaxis: 'Logical Lines of Code'

                plot csvFileName: 'phploc-functions.csv',
                    csvSeries: [[
                        file: 'build/logs/phploc.csv',
                        inclusionFlag: 'INCLUDE_BY_STRING',
                        exclusionValues: 'Functions,Named Functions,Anonymous Functions',
                        displayTableFlag: false,
                        url: ''
                    ]],
                    group: 'phploc', numBuilds: '100',
                    style: 'line', title: 'J - Types of Functions',
                    yaxis: 'Count'

                plot csvFileName: 'phploc-objects.csv',
                    csvSeries: [[
                        file: 'build/logs/phploc.csv',
                        inclusionFlag: 'INCLUDE_BY_STRING',
                        exclusionValues: 'Interfaces,Traits,Classes,Methods,Functions,Constants',
                        displayTableFlag: false,
                        url: ''
                    ]],
                    group: 'phploc', numBuilds: '100',
                    style: 'line', title: 'K - Structure Objects',
                    yaxis: 'Count'
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
