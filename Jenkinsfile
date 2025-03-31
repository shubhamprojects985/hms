pipeline {
    agent any

    environment {
        DEPLOY_PATH = "${isUnix() ? '/opt/lampp/htdocs/hms' : 'C:\\\\xampp\\\\htdocs\\\\hms'}" // Dedicated folder for HMS
        COMPOSER_CMD = "${isUnix() ? 'composer' : 'C:\\\\xampp\\\\php\\\\php.exe C:\\\\ProgramData\\\\ComposerSetup\\\\bin\\\\composer'}" // Flexible Composer path
        MYSQL_CMD = "${isUnix() ? 'mysql' : 'C:\\\\xampp\\\\mysql\\\\bin\\\\mysql.exe'}" // Flexible MySQL path
        SQL_FILE = 'hmisphp.sql' // Your SQL file
        DB_NAME = 'hmisphp' // Consistent database name
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/shubhamprojects985/hms.git',
                    credentialsId: 'github-credentials' // GitHub credentials
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    if (isUnix()) {
                        sh "${COMPOSER_CMD} install --no-dev --optimize-autoloader"
                    } else {
                        bat "${COMPOSER_CMD} install --no-dev --optimize-autoloader"
                    }
                }
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

        // stage('Run Tests') {
        //     steps {
        //         script {
        //             if (fileExists('vendor/bin/phpunit') || fileExists('vendor\\bin\\phpunit')) {
        //                 if (isUnix()) {
        //                     sh 'vendor/bin/phpunit --log-junit test-results.xml || true'
        //                 } else {
        //                     bat 'vendor\\bin\\phpunit --log-junit test-results.xml || exit 0'
        //                 }
        //             } else {
        //                 echo 'PHPUnit not found, skipping tests. Add tests to your project for better quality!'
        //             }
        //         }
        //     }
        // }

        stage('Deploy') {
            steps {
                script {
                    if (isUnix()) {
                        sh "mkdir -p ${DEPLOY_PATH} && rsync -av --exclude='.git' ./ ${DEPLOY_PATH}/"
                    } else {
                        bat "if not exist ${DEPLOY_PATH} mkdir ${DEPLOY_PATH} && robocopy . ${DEPLOY_PATH} /E /XD .git /MT"
                    }
                }
            }
        }

        stage('Setup Database') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'mysql-credentials', usernameVariable: 'DB_USER', passwordVariable: 'DB_PASS')]) {
                    script {
                        if (isUnix()) {
                            sh "${MYSQL_CMD} -u ${DB_USER} -p${DB_PASS} ${DB_NAME} < ${SQL_FILE} || echo 'DB setup might have issues, check logs!'"
                        } else {
                            bat "${MYSQL_CMD} -u ${DB_USER} -p${DB_PASS} ${DB_NAME} < ${SQL_FILE} || echo 'DB setup might have issues, check logs!'"
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline complete ho gaya bhai!'
        }
        failure {
            echo 'Kuch gadbad ho gaya, logs check kar le!'
        }
    }
}