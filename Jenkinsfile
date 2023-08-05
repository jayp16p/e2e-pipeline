pipeline{
    agent{
        label "jenkins-agent"
    }
    tools{
        jdk 'Java17'
        maven 'Maven3'
    }
    stages{
        stage("Cleanup Workspace"){
            steps{
                cleanWs()
            } 
        }

        stage("Get the code from github"){
            steps{
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/jayp16p/e2e-pipeline'
            }
    }

    stage("Build the application"){
            steps{
                sh "mvn clean package"
            }
    }

    stage("Test Application"){
            steps{
                sh "mvn test"
            }
    }

    stage("Sonarqube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
                        sh "mvn sonar:sonar \
                            -Dsonar.projectKey=demo \
                            -Dsonar.host.url=http://192.168.56.13:9000 \
                            -Dsonar.login=c2c05facb0f246112a54248ce3a686e3677d6007"
                    }
            }
    }
    }
}
}
