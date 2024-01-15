pipeline {
    agent any 
    
    tools{
        jdk 'jdk11'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    
    stages{
        
        stage("Git Checkout"){
            steps{
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/santoshbd67/Petclinic.git'
            }
        }
        
        stage("Compile"){
            steps{
                sh "mvn clean compile"
            }
        }
        
         stage("Test Cases"){
            steps{
                sh "mvn test"
            }
        }
        
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Petclinic \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Petclinic '''
    
                }
            }
        }
        
        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --format HTML ', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
         stage("Build"){
            steps{
                sh " mvn clean install"
            }
        }
        
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'dockerhub') {
                        
                        sh "docker build -t image1 ."
                        sh "docker tag image1 santoshbd67/pet-clinic123:latest "
                        sh "docker push santoshbd67/pet-clinic123:latest "
                    }
                }
            }
        }
        
        stage("TRIVY"){
            steps{
                sh " trivy image santoshbd67/pet-clinic123:latest"
            }
        }
       stage('Deploy to Tomcat') {
            steps {
                script {
                    // Specify the target directory
                    def targetDir = '/opt/apache-tomcat-9.0.65/webapps/'

                    // Create the target directory if it doesn't exist
                    sh "sudo mkdir -p $targetDir"

                    // Copy the WAR file to the target directory
                    sh "cp \$WORKSPACE/target/petclinic.war $targetDir"
                }
            }
        }             
    }
}
