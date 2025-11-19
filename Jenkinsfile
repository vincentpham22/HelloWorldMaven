pipeline {
    agent any



    // Vous pouvez décommenter ce bloc une fois le problème de connexion réseau réglé
    /*
    triggers {
        // Vérifie le SCM toutes les 5 minutes et lance SEULEMENT s'il y a un changement
        pollSCM('H/5 * * * *')
    }
    */

// ----------------------------------------------------
// ÉTAPE DE BUILD PRINCIPALE
// ----------------------------------------------------
    stages {
        stage('Checkout') {
            steps {
                // Cette étape utilise les identifiants pour cloner le dépôt
                git branch: 'master', 
                    credentialsId: 'github-credentials', 
                    url: 'https://github.com/vincentpham22/HelloWorldMaven.git' // ❗ Votre URL
                echo "Dépôt cloné."
            }
        }
        stage('Build') {
            steps {
                script {
                    // ❗ ATTENTION: Le 'git config' doit être fait ICI, car il sera appliqué
                    // au tag dans la section post/success
                    sh 'git config user.email "jenkins@votre-domaine.com"'
                    sh 'git config user.name "Jenkins CI"'

                    // 2. EXÉCUTER LE BUILD
                    sh 'mvn clean install'
                }
            }
        }
    } // <-- FERMETURE DU BLOC 'stages'
    // ----------------------------------------------------
// ÉTAPES POST-BUILD : TAGGING ET PUSH (Ce bloc est au même niveau que 'stages')
// ----------------------------------------------------
    post {

        // Ce bloc s'exécute si le stage principal a réussi
        success {
            script {
                def tagName = "V-2.${env.BUILD_NUMBER}"

                // Injection sécurisée des identifiants (token)
                withCredentials([
                    usernamePassword(
                        credentialsId: 'github-credentials', 
                        usernameVariable: 'GIT_USER', 
                        passwordVariable: 'GIT_TOKEN'
                    )
                ]) {

                    echo "Pipeline Réussi. Création et poussée du tag '${tagName}'."
                    sh "git tag ${tagName}"
                    // Construction de l'URL pour l'authentification Git
                    def gitRepoUrl = "https://github.com/vincentpham22/HelloWorldMaven.git" // ❗ MODIFIER CETTE URL
                    def gitPushUrl = "https://${GIT_USER}:${GIT_TOKEN}@${gitRepoUrl.split('//')[1]}"

                    sh "git push ${gitPushUrl} ${tagName}"
                }
            }
        }

        // Ce bloc s'exécute si le stage principal a échoué
        failure {
            script {
                // Tag de failure manquant, ajoutons-le pour être complet
                def tagName = "V-2.${env.BUILD_NUMBER}-failure"

                withCredentials([
                    usernamePassword(
                        credentialsId: 'github-credentials', 
                        usernameVariable: 'GIT_USER', 
                        passwordVariable: 'GIT_TOKEN'
                    )
                ]) {
                    echo "Pipeline Échoué. Création du tag de failure."
                    sh "git tag ${tagName}"

                    def gitRepoUrl = "https://github.com/vincentpham22/HelloWorldMaven.git"
                    def gitPushUrl = "https://${GIT_USER}:${GIT_TOKEN}@${gitRepoUrl.split('//')[1]}"
                    sh "git push ${gitPushUrl} ${tagName}"
                }
            }
        }
    } // <-- FERMETURE DU BLOC 'post'
} // <-- FERMETURE DU BLOC 'pipeline'
