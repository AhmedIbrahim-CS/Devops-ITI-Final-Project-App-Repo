pipeline {
   agent { label 'node' }  

    parameters {
        choice(name: 'ENV', choices: ['dev', 'test', 'prod', "release"])
    }
    stages {
        stage('build') {
            steps {
                script {
                    echo 'build'
                    if (params.ENV == "release") {
                        withCredentials([usernamePassword(credentialsId: 'docker-cred', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                            sh '''
                                docker login -u ${USERNAME} -p ${PASSWORD}
                                docker build -t ahmedibrahimcs/iti-app:v${BUILD_NUMBER} .
                                docker push ahmedibrahimcs/iti-app:v${BUILD_NUMBER}
                                echo ${BUILD_NUMBER} > ../build.txt
                                echo ${ENV}
                            '''
                        }
                    } else {
                        echo "user chose ${params.ENV}"
                    }
                }
            }
        }
        stage('deploy') {
            steps {
                echo 'deploy'
                script {
                    if (params.ENV == "dev" || params.ENV == "test" || params.ENV == "prod") {
                        withCredentials([file(credentialsId: 'kubeconfig-cred', variable: 'KUBECONFIG')]) {
                           sh '''
                                export BUILD_NUMBER=$(cat ../build.txt)
                                if helm status "release-$BUILD_NUMBER" --kubeconfig $KUBECONFIG --namespace iti >/dev/null 2>&1; then
                                    helm upgrade "release-$BUILD_NUMBER" ./Deployment --values Deployment/values.yaml --set image.tag=v$BUILD_NUMBER --set service.name=myservice$BUILD_NUMBER --set deployment.name=iti$BUILD_NUMBER --kubeconfig $KUBECONFIG --namespace iti
                                else
                                    helm install "release-$BUILD_NUMBER" ./Deployment --values Deployment/values.yaml --set image.tag=v$BUILD_NUMBER --set service.name=myservice$BUILD_NUMBER --set deployment.name=iti$BUILD_NUMBER --kubeconfig $KUBECONFIG --namespace iti
                                fi
                             '''
                        }
                    } else {
                        echo "user chose ${params.ENV}"
                    }
                }
            }
        }
    }
}

