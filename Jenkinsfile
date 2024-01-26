node {
    stage('Checkout') {
        checkout scmGit(
            branches         : [[name: '*/master']],
            extensions       : [],
            userRemoteConfigs: [[url: '/home/Docker/simple-java-maven-app']]
        )
    }
    withDockerContainer(
        image: 'maven:3.9.6-eclipse-temurin-17-alpine', 
        args : '--user=root -v /root/.m2:/root/.m2'
    ) {
        stage('Build') {
            sh 'mvn -B -DskipTests clean package'
        }
        try {
            stage('Test') {
                sh 'mvn test'
            } 
        } finally {
            junit 'target/surefire-reports/*.xml'
        }
        stage('Manual Approval') {
            input 'Lanjutkan ke tahap Deploy?'
        }
        stage('Deploy') {
            sh './jenkins/scripts/deliver.sh'
            sh 'apk add --no-cache openssh-client'
            
            def JAR_FILE       = sh(script: 'cd target && ls *.jar', returnStdout: true).trim()
            def DOCKER_COMMAND = "docker run --rm -v /home/ubuntu/${JAR_FILE}:/app/${JAR_FILE} maven:3.9.6-eclipse-temurin-17-alpine java -jar /app/${JAR_FILE}"

            withCredentials([file(credentialsId: 'ec2-private-key', variable: 'PRIVATE_KEY_FILE')]) {
                sh 'scp -o StrictHostKeyChecking=no -i ${PRIVATE_KEY_FILE} target/*.jar ubuntu@3.1.81.239:/home/ubuntu/'
                sh 'ssh -o StrictHostKeyChecking=no -i ${PRIVATE_KEY_FILE} ubuntu@3.1.81.239 ' + DOCKER_COMMAND
            }
            
            sleep 60
        }
    }
}

