pipeline {
    agent any

    environment {
        // Paths ko flexible banaya, apne setup ke hisaab se change kar lena
        DEPLOY_PATH = "${isUnix() ? '/opt/lampp/htdocs' : 'C:\\\\xampp\\\\htdocs'}" // Default paths, change kar sakta hai
        COMPOSER_CMD = 'composer'         // Composer command, agar alag hai to yahan set kar
        MYSQL_CMD = 'mysql'               // MySQL command, agar alag hai to yahan set kar
        SQL_FILE = 'hmisphp.sql'         // Apne SQL script ka naam daal dena
    }

   stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/shubhamprojects985/hms.git',
                    credentialsId: 'github-credentials' // Git credentials yahan add kiye
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    // Cross-platform composer install
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
                        // Config file dynamically banayenge
                        def configTemplate = '''
                            <?php
                            define('DB_HOST', 'localhost');
                            define('DB_USER', 'root');
                            define('DB_PASS', '');
                            define('DB_NAME', 'hmisphp');
                            ?>
                        '''
                        def finalConfig = configTemplate.replace('__DB_USER__', DB_USER).replace('__DB_PASS__', DB_PASS)
                        writeFile file: 'config.php', text: finalConfig
                    }
                }
            }
        }

        // stage('Run Tests') {
        //     steps {
        //         script {
        //             if (isUnix()) {
        //                 sh 'vendor/bin/phpunit --log-junit test-results.xml'
        //             } else {
        //                 bat 'vendor\\bin\\phpunit --log-junit test-results.xml'
        //             }
        //         }
        //     }
        // }

        stage('Deploy') {
            steps {
                script {
                    // Files deploy karenge aur DB setup karenge
                    if (isUnix()) {
                        sh "mkdir -p ${DEPLOY_PATH} && cp -rf . ${DEPLOY_PATH}/"
                        withCredentials([usernamePassword(credentialsId: 'mysql-credentials', usernameVariable: 'DB_USER', passwordVariable: 'DB_PASS')]) {
                            sh "${MYSQL_CMD} -u ${DB_USER} -p${DB_PASS} hospital_db < ${SQL_FILE}"
                        }
                    } else {
                        bat "if not exist ${DEPLOY_PATH} mkdir ${DEPLOY_PATH} && xcopy . ${DEPLOY_PATH} /E /I /Y"
                        withCredentials([usernamePassword(credentialsId: 'mysql-credentials', usernameVariable: 'DB_USER', passwordVariable: 'DB_PASS')]) {
                            bat "${MYSQL_CMD} -u ${DB_USER} -p${DB_PASS} hospital_db < ${SQL_FILE}"
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
            echo 'Kuch gadbad ho gaya, check kar le!'
        }
    }
}
