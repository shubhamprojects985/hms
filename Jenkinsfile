pipeline {
    agent any

    environment {
        DEPLOY_PATH = "${env.DEPLOY_PATH ?: 'C:\\xampp\\htdocs\\hms'}"
        SQL_FILE = "${env.SQL_FILE ?: 'hmisphp.sql'}" // Allow override if name differs
        DB_NAME = 'hmisphp'
        DB_HOST = "${env.DB_HOST ?: 'localhost'}"
        PHP_PATH = "${env.PHP_PATH ?: 'C:\\xampp\\php'}"
        COMPOSER_PATH = "${env.COMPOSER_PATH ?: 'C:\\ProgramData\\ComposerSetup\\bin'}"
        MYSQL_PATH = "${env.MYSQL_PATH ?: 'C:\\Program Files\\MySQL\\MySQL Server 8.0\\bin'}"
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Code checked out from SCM'
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    if (fileExists('composer.json')) {
                        if (isUnix()) {
                            sh 'composer install --no-dev --optimize-autoloader'
                        } else {
                            bat "\"${COMPOSER_PATH}\\composer.bat\" install --no-dev --optimize-autoloader"
                        }
                    } else {
                        echo 'No composer.json found, skipping dependency installation.'
                    }
                }
            }
        }

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

        stage('Deploy') {
            steps {
                script {
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
                        def mysqlCmd = isUnix() ? 'mysql' : "\"${MYSQL_PATH}\\mysql.exe\""
                        def createDbCommand = "${mysqlCmd} -u${DB_USER} -h${DB_HOST} -e \"CREATE DATABASE IF NOT EXISTS ${DB_NAME};\""
                        def importDbCommand = "${mysqlCmd} -u${DB_USER} -h${DB_HOST} ${DB_NAME} < ${SQL_FILE}"

                        // Check if SQL file exists before running
                        if (fileExists(SQL_FILE)) {
                            withEnv(["MYSQL_PWD=${DB_PASS}"]) {
                                if (isUnix()) {
                                    sh createDbCommand
                                    sh importDbCommand
                                } else {
                                    bat createDbCommand
                                    bat importDbCommand
                                }
                            }
                        } else {
                            error "SQL file '${SQL_FILE}' not found in workspace!"
                        }
                    }
                }
            }
        }
    }

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
