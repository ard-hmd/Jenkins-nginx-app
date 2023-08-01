pipeline {

    agent any // Jenkins will be able to select all available agents

    stages {

        stage(' Docker run -> Nginx Reverse Proxy'){ // run container from our builded image
            steps {
                script {
                sh '''
                docker rm -f nginx
                docker run -d --name nginx --network monReseau -p 8087:8087 -v ./nginx-app/nginx_config.conf:/etc/nginx/conf.d/default.conf nginx:latest
                sleep 10
                '''
                }
            }
        }

        stage('Test Acceptance -> Nginx Reverse Proxy'){ // we launch the curl command to validate that the container responds to the request
            steps {
                    script {
                    sh '''
                    curl -I -s http://localhost:8087/api/v1/movies/docs | grep HTTP/1.1 | awk '{print $2}'
                    curl -I -s http://localhost:8087/api/v1/casts/docs | grep HTTP/1.1 | awk '{print $2}'
                    '''
                    }
            }

        }

        stage('Deploiement en dev -> Nginx Reverse Proxy'){
            environment
            {
                KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
            }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp nginx-chart/values.yaml values.yaml
                cat values.yaml
                helm upgrade --install nginx-chart-release ./nginx-chart --values=values.yaml --set servicePorts.castService=8001 --set servicePorts.movieService=8002 --namespace dev
                '''
                }
            }

        }

        stage('Deploiement en staging -> Nginx Reverse Proxy'){
            environment
            {
                KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
            }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp nginx-chart/values.yaml values.yaml
                cat values.yaml
                helm upgrade --install nginx-chart-release ./nginx-chart --values=values.yaml --set servicePorts.castService=8003 --set servicePorts.movieService=8004 --namespace staging
                '''
                }
            }

        }

        stage('Deploiement en prod -> Nginx Reverse Proxy'){
            environment
            {
                KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
            }
            steps {
            // Create an Approval Button with a timeout of 15minutes.
            // this require a manuel validation in order to deploy on production environment
                    timeout(time: 15, unit: "MINUTES") {
                        input message: 'Do you want to deploy in production ?', ok: 'Yes'
                    }

                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp nginx-chart/values.yaml values.yaml
                cat values.yaml
                helm upgrade --install nginx-chart-release ./nginx-chart --values=values.yaml --set servicePorts.castService=8005 --set servicePorts.movieService=8006 --namespace prod
                '''
                }
            }
    }
    }
}
