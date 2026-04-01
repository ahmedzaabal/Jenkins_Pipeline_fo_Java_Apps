def COLOR_MAP = [
    SUCCESS: 'good',
    UNSTABLE: 'warning',
    FAILURE: 'danger',
    ABORTED: 'danger'
]

pipeline{
    agent any
    tools{
        maven 'Maven-3.9.9'
        jdk 'JDK17'
    }

    environment {
        registryCredential = 'Put your registry credentials here, for example: ecr:us-east-1:awscreds'
        imageName = "the image name from the ECR, for example:418272758310.dkr.ecr.us-east-1.amazonaws.com/<app-name>"
        Name_of_the_ECR = "The name of the ECR, for example: https://418272758310.dkr.ecr.us-east-1.amazonaws.com"
        cluster = 'name of the cluster in ECS'
        service = 'Name of the service inside the cluster'
    }

    stages{
        stage("Fetch Code"){
            steps{
                git branch: '<name of the branch>' , url: 'https://<the github repository url>'
            }
        }


        stage("Build"){
            steps{
                sh 'mvn install -DskipTests'
            }
            post {
                success {
                    echo "Build successful!"
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

        stage("Unit Test"){
            steps{
                sh 'mvn test'
            }
        }

        stage("Checkstyle Analysis"){
            steps{
                sh 'mvn checkstyle:check -Dcheckstyle.failOnViolation=false'
            }
        }

        stage("SonarQube Analysis"){
            environment {
                scannerHome = tool 'sonar6.2'
            }
            steps{
                withSonarQubeEnv('sonarserver') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey={kindly put your project name here} \
                    -Dsonar.projectName={here as well} \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/ \
                    -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
                    
                    '''
                }
            }
        }

        stage("Quality Gate"){
            steps{
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("Build Docker Image"){
            steps{
                script {
                    dockerImage = docker.build( imageName + ":$BUILD_NUMBER", "Here, you will need to mention the path to the Dockerfile, for example: ./Docker-files/app/multistage/Dockerfile")
                }
            }
        }

        stage ("Push Docker Image to ECR"){
            steps{
                script {
                    docker.withRegistry(Name_of_the_ECR, registryCredential) {
                        dockerImage.push("$BUILD_NUMBER")
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage ("Remove Container Images") {
            steps{
                sh 'docker rmi -f $(docker images -a -q)'
            }
        }

        stage ('Deploy to ECS') {
            steps {
                withAWS(credentials: 'awscreds', region: 'us-east-1') {
                    sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
                }
            }
        }

    }
}