pipeline {
    agent any

    // Environment variables for customization
    environment {
        DEPLOY_PATH = "${env.DEPLOY_PATH ?: (isUnix() ? '/var/www/html/hms' : 'C:\\xampp\\htdocs\\hms')}"
        SQL_FILE = 'hmisphp.sql'
        DB_NAME = 'hmisphp'
        DB_HOST = "${env.DB_HOST ?: 'localhost'}"
    }

    stages {
        // Stage 1: Checkout code from GitHub
        stage('Checkout') {
            steps {
                script {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: '*/main']],
                        userRemoteConfigs: [[
                            url: 'https://github.com/shubhamprojects985/hms.git',
                            credentialsId: 'github-credentials'
                        ]]
                    ])
                }
            }
        }

        // Stage 2: Install PHP dependencies with Composer
        stage('Install Dependencies') {
            steps {
                script {
                    if (fileExists('composer.json')) {
                        if (isUnix()) {
                            sh 'composer install --no-dev --optimize-autoloader'
                        } else {
                            bat 'composer install --no-dev --optimize-autoloader'
                        }
                    } else {
                        echo 'No composer.json found, skipping dependency installation.'
                    }
                }
            }
        }

        // Stage 3: Set up configuration file with MySQL credentials
        stage('Setup Config') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'mysql-credentials',
                    usernameVariable: 'DB_USER',
                    passwordVariable: 'DB_PASS'
                )]) {
                    script {
                        def configTemplate = '''
                            <?php
                            define('DB_HOST', '__DB_HOST__');
                            define('DB_USER', '__DB_USER__');
                            define('DB_PASS', '__DB_PASS__');
                            define('DB_NAME', '__DB_NAME__');
                            ?>
                        '''
                        def finalConfig = configTemplate.replace('__DB_HOST__', DB_HOST)
                                                        .replace('__DB_USER__', DB_USER)
                                                        .replace('__DB_PASS__', DB_PASS)
                                                        .replace('__DB_NAME__', DB_NAME)
                        writeFile file: 'config.php', text: finalConfig
                    }
                }
            }
        }

        // Stage 4: Deploy application and set up database
        stage('Deploy') {
            steps {
                script {
                    // Copy files to deployment path
                    if (isUnix()) {
                        sh "mkdir -p ${DEPLOY_PATH} && cp -r . ${DEPLOY_PATH}"
                    } else {
                        bat "if not exist \"${DEPLOY_PATH}\" mkdir \"${DEPLOY_PATH}\" && xcopy . \"${DEPLOY_PATH}\" /E /I /Y"
                    }
                }
                withCredentials([usernamePassword(
                    credentialsId: 'mysql-credentials',
                    usernameVariable: 'DB_USER',
                    passwordVariable: 'DB_PASS'
                )]) {
                    script {
                        // Define MySQL command based on OS
                        def mysqlCmd
                        if (isUnix()) {
                            mysqlCmd = 'mysql'
                        } else {
                            def defaultMysqlPath = 'C:\\Program Files\\MySQL\\MySQL Server 8.0\\bin'
                            mysqlCmd = "${env.MYSQL_PATH ?: defaultMysqlPath}\\mysql.exe"
                        }

                        // Create database and import SQL file
                        def createDbCommand = "${mysqlCmd} -u${DB_USER} --password=${DB_PASS} -h${DB_HOST} -e \"CREATE DATABASE IF NOT EXISTS ${DB_NAME};\""
                        def importDbCommand = "${mysqlCmd} -u${DB_USER} --password=${DB_PASS} -h${DB_HOST} ${DB_NAME} < ${SQL_FILE}"

                        if (isUnix()) {
                            sh createDbCommand
                            sh importDbCommand
                        } else {
                            bat createDbCommand
                            bat importDbCommand
                        }
                    }
                }
            }
        }
    }

    // Post-build actions
    post {
        always {
            echo 'Pipeline execution completed!'
        }
        success {
            echo 'Pipeline succeeded! HMS is deployed.'
        }
        failure {
            echo 'Pipeline failed! Check the logs for details.'
        }
    }
}