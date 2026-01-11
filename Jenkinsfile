// Jenkinsfile - Pipeline CI/CD pour microservices
// Ce pipeline automatise: clonage, build Maven, analyse SonarQube, déploiement Docker
//
// Prérequis dans Jenkins:
// - Installation Maven nommée 'maven' (Manage Jenkins → Tools)
// - SonarQube Scanner installé (Manage Jenkins → Tools)
// - Serveurs SonarQube configurés: 'SonarQube-Car' et 'SonarQube-Client' (Manage Jenkins → System)
//
// Note: Ce script est adapté pour Linux/Mac (utilise 'sh' au lieu de 'bat')
// Pour Windows, remplacer 'sh' par 'bat' et adapter les chemins

pipeline {
    // Exécution sur n'importe quel agent disponible
    agent any

    // Outils nécessaires pour le pipeline
    tools {
        // Maven doit être configuré dans Jenkins avec le nom exact 'maven'
        maven 'maven'
    }

    // Définition des étapes du pipeline
    stages {

        // ÉTAPE 1: Clonage du dépôt GitHub
        stage('Cloner le dépôt') {
            steps {
                echo '🔄 Clonage du dépôt GitHub...'
                script {
                    // Clonage de la branche main du dépôt
                    // Remplacez l'URL par votre dépôt GitHub
                    git branch: 'main', 
                         url: 'https://github.com/ZouizzaKhalil/jenkins.git',
                         credentialsId: '' // Optionnel: ID des credentials GitHub si dépôt privé
                }
            }
        }

        // ÉTAPE 2: Build et analyse SonarQube (exécution en parallèle pour performance)
        stage('Build and SonarQube Analysis') {
            parallel {
                
                // === MICROSERVICE CAR ===
                stage('Car Service') {
                    stages {
                        // 2.1. Build du service Car
                        stage('Build Car Service') {
                            steps {
                                dir('car') {
                                    echo '🔨 Compilation et génération du service Car...'
                                    script {
                                        // Compilation Maven avec saut des tests (optionnel)
                                        sh 'mvn clean install -DskipTests'
                                    }
                                }
                            }
                        }

                        // 2.2. Analyse SonarQube du service Car
                        stage('SonarQube Analysis Car Service') {
                            steps {
                                dir('car') {
                                    script {
                                        // Récupération du chemin Maven configuré dans Jenkins
                                        def mvn = tool 'maven'
                                        
                                        // Injection des variables d'environnement SonarQube
                                        // Le nom 'SonarQube-Car' doit correspondre à la config dans Jenkins
                                        withSonarQubeEnv('SonarQube-Car') {
                                            // Exécution de l'analyse SonarQube via plugin Maven
                                            // Les paramètres -Dsonar.* configurent l'analyse
                                            sh "${mvn}/bin/mvn clean verify " +
                                               "sonar:sonar " +
                                               "-Dsonar.projectKey=car " +
                                               "-Dsonar.projectName='car' " +
                                               "-DskipTests"
                                        }
                                    }
                                }
                            }
                        }
                    }
                }

                // === MICROSERVICE CLIENT ===
                stage('Client Service') {
                    stages {
                        // 2.3. Build du service Client
                        stage('Build Client Service') {
                            steps {
                                dir('client') {
                                    echo '🔨 Compilation et génération du service Client...'
                                    script {
                                        sh 'mvn clean install -DskipTests'
                                    }
                                }
                            }
                        }

                        // 2.4. Analyse SonarQube du service Client
                        stage('SonarQube Analysis Client Service') {
                            steps {
                                dir('client') {
                                    script {
                                        def mvn = tool 'maven'
                                        
                                        // Injection des variables d'environnement SonarQube
                                        // Le nom 'SonarQube-Client' doit correspondre à la config dans Jenkins
                                        withSonarQubeEnv('SonarQube-Client') {
                                            sh "${mvn}/bin/mvn clean verify " +
                                               "sonar:sonar " +
                                               "-Dsonar.projectKey=client " +
                                               "-Dsonar.projectName='client' " +
                                               "-DskipTests"
                                        }
                                    }
                                }
                            }
                        }
                    }
                }

                // === MICROSERVICE GATEWAY ===
                // Note: Gateway est buildé mais pas analysé par SonarQube (ajoutable si besoin)
                stage('Gateway Service') {
                    steps {
                        dir('gateway') {
                            echo '🔨 Compilation et génération du service Gateway...'
                            script {
                                sh 'mvn clean install -DskipTests'
                            }
                        }
                    }
                }

                // === SERVEUR EUREKA ===
                // Note: Eureka est buildé mais pas analysé par SonarQube (ajoutable si besoin)
                stage('Eureka Server') {
                    steps {
                        dir('server_eureka') {
                            echo '🔨 Compilation et génération du serveur Eureka...'
                            script {
                                sh 'mvn clean install -DskipTests'
                            }
                        }
                    }
                }
            }
        }

        // ÉTAPE 3: Déploiement avec Docker Compose
        stage('Docker Compose') {
            steps {
                dir('deploy') {
                    echo '🐳 Création et déploiement des conteneurs Docker...'
                    script {
                        // Rebuild et redémarrage des services en mode détaché
                        // -d: mode détaché (background)
                        // --build: force le rebuild des images
                        sh 'docker-compose up -d --build'
                    }
                }
            }
        }
    }

    // Actions post-exécution (optionnel)
    post {
        // En cas de succès
        success {
            echo '✅ Pipeline exécuté avec succès!'
            // Optionnel: notification (email, Slack, etc.)
        }
        // En cas d'échec
        failure {
            echo '❌ Pipeline échoué. Consultez les logs pour plus de détails.'
        }
        // Dans tous les cas
        always {
            echo '📋 Nettoyage et finalisation...'
            // Optionnel: nettoyage des artifacts temporaires
        }
    }
}
