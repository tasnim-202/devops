pipeline {
    agent any

    triggers {
        // Webhook GitHub : build lance a chaque push
        githubPush()
        // Secours si le webhook ne joint pas Jenkins
        pollSCM('H/5 * * * *')
    }

    options {
        // Le checkout est fait explicitement dans le stage 1
        skipDefaultCheckout(true)
        disableConcurrentBuilds()
    }

    stages {
        stage('Checkout') {
            steps {
                // Utilise le depot, la branche et les credentials configures dans le job
                checkout scm
            }
        }
        stage('Build Maven') {
            steps {
                dir('backend') {
                    // Maven Wrapper = mvn compile, sans installer Maven ; genere backend/target
                    bat 'call .\\mvnw.cmd -B clean compile'
                }
            }
        }
    }

    post {
        success { echo 'Pipeline termine avec succes' }
        failure { echo 'Pipeline en echec : voir les logs ci-dessus' }
    }
}
