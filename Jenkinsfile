pipeline {
    agent any
    
    tools 
    {
         maven 'Maven3'
    }

    //CI build starts here..
    stages {
        stage ("Build") {
            steps {
                sh "mvn clean install -f MyWebApp/pom.xml"
            }
        }
    
        stage ("Coverage") {
            steps {
                jacoco()
            }
        }
        
        stage ("Code scan") {
            steps {
                withSonarQubeEnv("SonarQube") {
                    sh "mvn sonar:sonar -f MyWebApp/pom.xml"
                }
            }
        }
        
         stage("Quality Gate") {
            steps
            {
                timeout(time: 1, unit: 'HOURS') {
                   waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage ("Nexus upload") {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'MyWebApp', classifier: '', file: 'MyWebApp/target/MyWebApp.war', type: 'war']], credentialsId: '114f9756-f82f-4ac6-9057-40f628e03525', groupId: 'cac', nexusUrl: 'ec2-54-208-154-173.compute-1.amazonaws.com:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '1.0-SNAPSHOT'
            }
        }
        
        stage ("DEV deploy") {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'be668c39-9d6e-4b92-ba3b-7136395b9f60', path: '', url: 'http://ec2-3-88-182-162.compute-1.amazonaws.com:8080/')], contextPath: null, war: '**/*.war'
            }
        }
        
        stage ("DEV notify") {
            steps {
                slackSend channel: 'feb-2024-weekday-batch', message: 'DEV Deployment was done, please start testing in DEV env'
            }
        }
        
        //CI build stops here..
        
        /*
        * CD pipeline starts here
        */
        
           stage ('DEV Approve') {
                steps {
                    echo "Taking approval from DEV Manager for QA Deployment"
                    timeout(time: 7, unit: 'DAYS') {
                    input message: 'Do you want to deploy?', submitter: 'admin'
                    }
                }
             }

        stage ("QA deploy") {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'be668c39-9d6e-4b92-ba3b-7136395b9f60', path: '', url: 'http://ec2-3-88-182-162.compute-1.amazonaws.com:8080/')], contextPath: null, war: '**/*.war'
            }
        }
        
        stage ("QA notify") {
            steps {
                slackSend channel: 'feb-2024-weekday-batch,qa-testing-team' , message: 'QA Deployment was done, please start testing in QA env'
            }
        }
        
        
         stage ('QA Approve') {
                steps {
                    echo "Taking approval from QA Manager for PROD Deployment"
                    timeout(time: 7, unit: 'DAYS') {
                    input message: 'Do you want to deploy?', submitter: 'admin'
                    }
                }
             }

        stage ("PROD deploy") {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'be668c39-9d6e-4b92-ba3b-7136395b9f60', path: '', url: 'http://ec2-3-88-182-162.compute-1.amazonaws.com:8080/')], contextPath: null, war: '**/*.war'
            }
        }
        
        stage ("final notify") {
            steps {
                slackSend channel: 'feb-2024-weekday-batch, qa-testing-team, product-owners-teams' , message: 'PROD Deployment was done, please inform end customers'
            }
        }
    }
}
