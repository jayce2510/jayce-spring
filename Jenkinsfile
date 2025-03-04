pipeline {

    agent any

    environment {
        MYSQL_ROOT_LOGIN = credentials('mysql-root')
    }

    stages {

        stage('Packaging/Pushing image') {
            steps {
                withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                    sh 'docker build -t khaliddinh/springboot .'
                    sh 'docker push alviss2510/springboot'
                }
            }
        }

        stage('Deploy MySQL to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull mysql:8.0'
                sh 'docker network create dev || echo "this network exists"'
                sh 'docker container stop khalid-mysql || echo "this container does not exist" '
                sh 'echo y | docker container prune '
                sh 'docker volume rm khalid-mysql-data || echo "no volume"'

                sh "docker run --name khalid-mysql --rm --network dev -v khalid-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_LOGIN_PSW} -e MYSQL_DATABASE=db_example -d mysql:8.0"
                sh 'sleep 15'
                sh "docker exec -i khalid-mysql mysql --user=root --password=${MYSQL_ROOT_LOGIN_PSW} < script"
            }
        }

        stage('Deploy Spring Boot to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull alviss2510/springboot'
                sh 'docker network create dev || echo "this network exists"'
                sh 'docker container stop jayce-springboot || echo "this container does not exist" '
                sh 'echo y | docker container prune '

                sh "docker container run --name jayce-springboot --rm --network dev -p 8081:8080 -d alviss2510/springboot"
            }
        }
 
    }
    post {
        // Clean after build
        always {
            cleanWs()
        }
    }
}
