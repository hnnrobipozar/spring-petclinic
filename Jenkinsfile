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
                    docker.build("spring-build", "-f Dockerfile_build .")
                }
            }
        }

        stage('3. Testowanie aplikacji') {
            steps {
                script {
                    docker.build("spring-test", "-f Dockerfile_test .")
                }
            }
        }

        stage('4. Deploy kontenerów') {
            steps {
                script {
                    try {
                        def TIMESTAMP = sh(script: 'date +%Y%m%d%H%M%S', returnStdout: true).trim()
                        env.TIMESTAMP = TIMESTAMP
                        def buildContainerId = docker.build("spring-build", "-f Dockerfile_build .").id
                        def testContainerId = docker.build("spring-test", "-f Dockerfile_test .").id
                        sh "docker run -d --name build${TIMESTAMP} ${buildContainerId}"
                        sh "docker run -d --name test${TIMESTAMP} ${testContainerId}"
                        sh "docker logs build${TIMESTAMP} > build_log.txt"
                        sh "docker logs test${TIMESTAMP} > test_log.txt"
                        sh "docker stop build${TIMESTAMP} test${TIMESTAMP}"
                        sh "tar -czf Artifact_${TIMESTAMP}.tar.gz build_log.txt test_log.txt"
                        archiveArtifacts artifacts: "Artifact_${TIMESTAMP}.tar.gz", onlyIfSuccessful: true
                    } 
                    catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        error "Błąd podczas deployu: ${e.message}"
                    }
                }
            }
        }
    }
}
