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
        args : '-v /root/.m2:/root/.m2'
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
    }
}

