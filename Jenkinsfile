pipeline {
    agent any
    environment {
                PROJECT_ID = 'phonic-skyline-364017'
                CLUSTER_NAME = 'cluster-two'
                LOCATION = 'us-central1-c'
                CREDENTIALS_ID = 'my-project'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build image') {
            steps {
                script {
                    app = docker.build("nachorebella/pipeline:${env.BUILD_ID}")
                    }
            }
        }
        
        stage('Push image') {
            steps {
                script {
                    withCredentials([[$class: 'UsernamePasswordMultiBinding', \
                                credentialsId: 'dockerhub', \
                                usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {

                                sh "docker login -u $USERNAME -p $PASSWORD"
                         }
                        app.push("${env.BUILD_ID}")
			app.push('latest')
                        }      
                    }
                 }
    
        stage('Deploy to K8s') {
            steps{
                echo "Deployment started ..."
                sh 'ls -ltr'
                sh 'pwd'
                sh "sed -i 's/pipeline:latest/pipeline:${env.BUILD_ID}/g' deployment.yaml"
                step([$class: 'KubernetesEngineBuilder', \
                  projectId: env.PROJECT_ID, \
                  clusterName: env.CLUSTER_NAME, \
                  location: env.LOCATION, \
                  manifestPattern: 'deployment.yaml', \
                  credentialsId: env.CREDENTIALS_ID, \
                  verifyDeployments: true])
                } 
            }
        }    
}

