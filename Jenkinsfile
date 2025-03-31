pipeline {
    agent {
        label 'windows'
    }

    environment {
        DEPLOY_PATH = 'C:\\xampp\\htdocs\\hms'
        COMPOSER_CMD = 'C:\\ProgramData\\ComposerSetup\\bin\\composer.bat'
        MYSQL_CMD = 'C:\\xampp\\mysql\\bin\\mysql.exe'
        SQL_FILE = 'hmisphp.sql'
        DB_NAME = 'hmisphp'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/shubhamprojects985/hms.git', credentialsId: 'github-credentials'
            }
        }

        stage('Install Dependencies') {
            steps {
                bat "C:\\ProgramData\\ComposerSetup\\bin\\composer.bat install --no-dev --optimize-autoloader"
                // Or use: bat "${COMPOSER_CMD} install --no-dev --optimize-autoloader" if using the env variable
            }
        }

        stage('Setup Config') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'mysql-credentials', usernameVariable: 'DB_USER', passwordVariable: 'DB_PASS')]) {
                    script {
                        def configTemplate = '''
                            <?php
                            define('DB_HOST', 'localhost');
                            define('DB_USER', '__DB_USER__');
                            define('DB_PASS', '__DB_PASS__');
                            define('DB_NAME', '__DB_NAME__');
                            ?>
                        '''
                        def finalConfig = configTemplate.replace('__DB_USER__', DB_USER)
                                                        .replace('__DB_PASS__', DB_PASS)
                                                        .replace('__DB_NAME__', DB_NAME)
                        writeFile file: 'config.php', text: finalConfig
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                bat "if not exist ${DEPLOY_PATH} mkdir ${DEPLOY_PATH} && xcopy . ${DEPLOY_PATH} /E /I /Y"
                withCredentials([usernamePassword(credentialsId: 'mysql-credentials', usernameVariable: 'DB_USER', passwordVariable: 'DB_PASS')]) {
                    bat "echo CREATE DATABASE IF NOT EXISTS ${DB_NAME}; | ${MYSQL_CMD} -u ${DB_USER} -p${DB_PASS}"
                    bat "${MYSQL_CMD} -u ${DB_USER} -p${DB_PASS} ${DB_NAME} < ${SQL_FILE}"
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline complete!'
        }
        failure {
            echo 'Something went wrong, check the logs!'
        }
    }
}