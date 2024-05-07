pipeline {
    agent any

    stages {
        stage('1. Pobieranie repo') {
            steps {
                git branch: "main", url: "https://github.com/hnnrobipozar/spring-petclinic"
                sh 'git config user.email "adamhyzy2001@wp.pl"'
                sh 'git config user.name "hnnrobipozar"'
            }
        }

        stage('2. Budowanie aplikacji') {
            steps {
                script {
                    docker.build("petclinic-app:${env.BUILD_NUMBER}", "-f Dockerfile_build .")
                }
            }
        }

        stage('3. Testowanie aplikacji') {
            steps {
                script {
                   docker.build("petclinic-test:${env.BUILD_NUMBER}", "-f Dockerfile_test .")
                }
            }
        }

        stage('4. Deploy & publish') {
            steps {
                script {
                    try {
                      
                        
                        // Usuń istniejący kontener o nazwie "petclinic-app", jeśli istnieje 
                        sh "docker rm -f petclinic-app || true"
                        
                        // Uruchomienie kontenera z aplikacją
                        docker.image("petclinic-app:${env.BUILD_NUMBER}").run("-d --name petclinic-app -p 8080:8080")

                        // Smoke testy
                        sh "curl -s -o /dev/null -w '%{http_code}' http://localhost:8080/"
                        
                        // Eksportuj kontener do pliku tar
                        sh "docker save petclinic-app:${env.BUILD_NUMBER} -o petclinic-app-${env.BUILD_NUMBER}.tar"
        
                        // Archiwizacja artefaktu
                        archiveArtifacts artifacts: "petclinic-app-${env.BUILD_NUMBER}.tar", onlyIfSuccessful: true

                        // Publikacja artefaktu
                        stash includes: "petclinic-app-${env.BUILD_NUMBER}.tar", name: "petclinic-app-${env.BUILD_NUMBER}"
                    } 
                    catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        error "Błąd podczas deployu: ${e.message}"
                    }
                }
            }
        }
    }
}//komentarz
