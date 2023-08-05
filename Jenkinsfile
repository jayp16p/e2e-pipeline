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
                cleanws()
            } 
        }

        stage("Get the code from github"){
            steps{
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/jayp16p/e2e-pipeline'
            }
    }
    
}
}