pipeline{
    agent any

    stages {

        
        stage('Checkout') {
            steps {
                echo 'Checkout stage'
                git 'https://github.com/therealdevito/multi-k8s.git'
            }
        }

        stage('Docker login') {
            steps {
                echo 'Docker login'
                sh 'docker logout'
                sh 'echo "$DOCKER_TOKEN" | docker login -u "$DOCKER_USERNAME" --password-stdin'
            }
        } 

        stage('Build') {
            steps {
                echo 'Build Stage'
                sh 'docker build -t therealdevito/react-test -f ./client/Dockerfile.dev ./client'
            }
        }

        stage('Tests') {
            steps {
                echo 'Running tests'
                sh 'docker run therealdevito/react-test npm test -- --coverage'
            }
        }
        

        stage('Deploy') {
            steps {
                
                sh 'docker build -t therealdevito/multi-client:latest -f ./client/Dockerfile ./client'
                sh 'docker build -t therealdevito/multi-server:latest -f ./server/Dockerfile ./server' 
                sh 'docker build -t therealdevito/multi-worker:latest -f ./worker/Dockerfile ./worker' 

                sh 'docker push therealdevito/multi-client:latest' 
                sh 'docker push therealdevito/multi-server:latest'
                sh 'docker push therealdevito/multi-worker:latest'
                
                sh 'kubectl --kubeconfig=config config set-context --current --user=jenkins-sa'
                sh 'kubectl --kubeconfig=config version'
              
                sh 'kubectl --kubeconfig=config apply -f k8s'
                sh 'kubectl --kubeconfig=config set image deployments/server-deployment server=therealdevito/multi-server:latest'
                sh 'kubectl --kubeconfig=config set image deployments/client-deployment client=therealdevito/multi-client:latest'
                sh 'kubectl --kubeconfig=config set image deployments/worker-deployment worker=therealdevito/multi-worker:latest'
                
                
            }
        }
    }
}
