// Compatible agents Windows (bat) et Linux (sh)
def runCmd(String cmd) {
    if (isUnix()) {
        sh cmd
    } else {
        bat cmd
    }
}

def mvnw(String goals) {
    // Maven Wrapper : pas besoin d'installer Maven sur l'agent
    runCmd((isUnix() ? 'sh ./mvnw' : 'call .\\mvnw.cmd') + ' -B ' + goals)
}

pipeline {
    agent any

    triggers {
        // Verifie GitHub toutes les minutes et lance un build s'il y a un nouveau push
        pollSCM('* * * * *')
    }

    options {
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 30, unit: 'MINUTES')
    }

    environment {
        CI = 'true'
        NG_CLI_ANALYTICS = 'false'
    }

    stages {
        stage('Build Backend') {
            steps {
                dir('backend') {
                    mvnw('clean package -DskipTests')
                }
            }
        }
        stage('Test Backend') {
            steps {
                dir('backend') {
                    mvnw('test')
                }
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'backend/target/surefire-reports/*.xml'
                }
            }
        }
        stage('Install Frontend') {
            steps {
                dir('frontend') {
                    runCmd('npm ci')
                }
            }
        }
        stage('Build Frontend') {
            steps {
                dir('frontend') {
                    runCmd('npm run build')
                }
            }
        }
        stage('Test Frontend') {
            steps {
                dir('frontend') {
                    runCmd('npm test -- --watch=false')
                }
            }
        }
        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'backend/target/*.jar, frontend/dist/**', fingerprint: true
            }
        }
    }

    post {
        success { echo 'Pipeline termine avec succes' }
        failure { echo 'Pipeline en echec : voir les logs ci-dessus' }
    }
}
