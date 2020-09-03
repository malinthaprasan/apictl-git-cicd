pipeline {

    agent {
        node {
            label 'master'
        }
    }
    environment { 
        PATH = "/root/apictl:/root/.nvm/versions/node/v12.18.1/bin:$PATH"
    }
    options {
        buildDiscarder logRotator( 
                    daysToKeepStr: '16', 
                    numToKeepStr: '10'
            )
    }

    stages {

        stage('Setup Environments for APICTL') {
            steps {
                sh """
                ENVCOUNT=\$(apictl list envs --format {{.}} | wc -l)
                if [ "\$ENVCOUNT" == "0" ]; then
                    apictl add-env -e live --apim https://localhost:9443
                fi
                """
            }
        }

        stage('Deploy MobileStore API To Dev') {
            steps {
                sh """
                apictl login live -u admin -p admin -k
                apictl import-api -e live -f MobileStore-v1.0  -k --update -k
                sleep 6
                """
            }
        }

        stage('Test MobileStore API In Dev') {
            steps {
                sh """
                export TOKEN=\$(apictl get-keys -n MobileStore -v 1.0 -e dev -k)
                echo \$(envsubst < Testing/envs/dev.json) > Testing/envs/env.tmp.json
                newman run Testing/TestAPI.postman_collection.json -e Testing/envs/env.tmp.json --insecure
                """
            }
        }

        stage('Deploy MobileStore API To Prod') {
            steps {
                sh """
                apictl login prod -u admin -p admin -k
                apictl import-api -e prod -f MobileStore-v1.0  -k --update -k
                sleep 6
                """
            }
        }

        stage('Test MobileStore API In Prod') {
            steps {
                sh """
                export TOKEN=\$(apictl get-keys -n MobileStore -v 1.0 -e prod -k)
                echo \$(envsubst < Testing/envs/prod.json) > Testing/envs/env.tmp.json
                newman run Testing/TestAPI.postman_collection.json -e Testing/envs/env.tmp.json --insecure
                """
            }
        }

    }   
}