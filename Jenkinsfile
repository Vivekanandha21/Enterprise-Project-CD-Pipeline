pipeline {
    agent any

    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'v1', description: 'Docker image tag to deploy')
    }

    environment {
        IMAGE_TAG = "${params.IMAGE_TAG}"
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Vivekanandha21/Enterprise-Project-CD-Pipeline.git'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig(
                    credentialsId: 'kn8s-cred',
                    clusterName: 'capstone-cluster',
                    namespace: 'webapps',
                    restrictKubeConfigAccess: false,
                    serverUrl: 'https://C07D0456DBFB58D2747597FC36D535B9.gr7.ap-south-1.eks.amazonaws.com'
                ) {
                    echo "Deploying image with tag: ${IMAGE_TAG}"

                    sh '''
                        sed -i "s|image: vktechie21/bankapp:.*|image: vktechie21/bankapp:${IMAGE_TAG}|" kubernetes/Manifest.yaml
                        kubectl apply -f kubernetes/Manifest.yaml -n webapps
                        kubectl apply -f kubernetes/HPA.yaml -n webapps

                        echo "Waiting for pods to be ready..."
                        kubectl wait --for=condition=Ready pod -l app=bankapp -n webapps --timeout=180s

                        echo "Checking rollout status..."
                        kubectl rollout status deployment/bankapp -n webapps --timeout=120s
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl get pods -n webapps
                    kubectl get svc -n webapps
                '''
            }
        }
    }

    post {
        success {
            echo "CD pipeline completed successfully. BankApp deployed with tag ${IMAGE_TAG}."
        }
        failure {
            echo "CD pipeline failed. Please check deployment logs."
        }
    }
}

