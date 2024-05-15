pipeline {
    agent any
    tools {
        nodejs 'nodejs'
    }
    environment {
        SCANNER_HOME = tool 'sonarqube-scanner'
    }
    stages{
        stage("checkout git"){
            steps{
                git branch: 'main', url: 'https://github.com/AbderrazakBouasker/expensse-tracker'
            }
        }
        stage("sonarqube analyse"){
            steps{
                withSonarQubeEnv('sq'){
                    sh'''
                    $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=expense_tracker -Dsonar.projectKey=expense_tracker
                    '''
                }
            }
        }
        stage("installing dependency") {
            steps {
                sh "npm install"
            }
        }
        stage("build"){
            steps{
                script {
                    docker.build("abderrazakbouasker/expense-tracker:${env.BUILD_NUMBER}")
                }
            }
            }
            stage("push"){
                steps{
                    script {
                        docker.withRegistry('https://registry.hub.docker.com', 'docker-credentials') {
                            docker.image("abderrazakbouasker/expense-tracker:${env.BUILD_NUMBER}").push()
                            docker.image("abderrazakbouasker/expense-tracker:${env.BUILD_NUMBER}").push("latest")
                        }
                }
            }
            stage("deploy"){
                steps{
                    script{
                        withCredentials([file(credentialsId: 'k8s'),variable:'KUBECONFIG']){
                            sh'''
                                export KUBECONFIG = $KUBECONFIG_FILE
            					kubectl apply -f kubernetes/deployment.yaml
            					kubectl apply -f kubernetes/service.yaml
            					kubectl get svc
            					kubectl all
            					kubectl set image deployment/app app=$DOCKER_IMAGE
                            '''
                        }
                    }
                }
            }
        }
    }
}
