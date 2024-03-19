
pipeline {
    agent any

    environment {
            registryURI         = 'registry.hub.docker.com/'

            dev_registry        = 'asadis7171/dev'
            qa_registry         = 'asadis7171/qa'
            stage_registry      = 'asadis7171/stage'
            prod_registry       = 'asadis7171/prod'

            dev_dh_creds        = 'docker-cred'
            qa_dh_creds         = 'dh_cred_qa'
            stage_dh_creds      = 'dh_cred_stage'
            prod_dh_creds       = 'dh_cred_prod'

            COMMITID           = "${params.commit_id}"
        }

        parameters {
            choice(name: 'account', choices: ['dev', 'qa', 'stage', 'prod'], description: 'Select the environment.')
            string(name: 'commit_id', defaultValue: 'latest', description: 'provide commit id.')
        }

    stages {
        stage('Docker Image Build IN Dev') {
            when {
                    expression {
                        params.account == 'dev'
                }
            }

            environment {
                    dev_registry_endpoint = 'https://' + "${env.registryURI}" + "${env.dev_registry}"
                    dev_image             = "${env.dev_registry}" + ":$GIT_COMMIT"
                }

            steps {
                echo "Building Docker Image Logging in to Docker Hub & Pushing the Image" 
                script {
                    def app = docker.build( dev_image )
                    docker.withRegistry( dev_registry_endpoint, dev_dh_creds ) {
                    app.push()
                    }
                }
            }
                post {
                    always {
                        sh 'echo Cleaning docker Images from Jenkins.'
                        sh "docker rmi ${env.dev_image}"
                    }
                }
            }
        }
        stage('Pull Tag push to QA') {
            when {
                    expression {
                        params.account == 'qa'
                    }
            }

            environment {
                        dev_registry_endpoint = 'https://' + "${env.registryURI}" + "${env.dev_registry}"
                        qa_registry_endpoint  = 'https://' + "${env.registryURI}" + "${env.qa_registry}"
                        dev_image             = "${env.registryURI}" + "${env.dev_registry}" + ':' + "${env.COMMITID}"
                        qa_image              = "${env.registryURI}" + "${env.qa_registry}" + ':' + "${env.COMMITID}"
                    }
            steps {
                script {
                    docker.withRegistry(dev_registry_endpoint, dev_dh_creds) {
                            docker.image(dev_image).pull()
                    }
                
                sh 'echo Image pulled from DEV'
                sh 'echo Taggig Docker image from Dev to QA'
                sh "docker tag ${env.dev_image} ${env.qa_image}"
                
                 docker.withRegistry(qa_registry_endpoint , qa_dh_creds) {
                        docker.image(env.qa_image).push()
                 }
                
                sh 'echo Image pushed'
                 }
             }
             post {
                        always {
                            sh 'echo Cleaning docker Images from Jenkins.'
                            sh "docker rmi ${env.dev_image}"
                            sh "docker rmi ${env.qa_image}"
                        }
                    }

    }
        stage('Pull Tag push to stage') {
            when {
                    expression {
                        params.account == 'stage'
                    }
            }
              environment {
                        qa_registry_endpoint        = 'https://' + "${env.registryURI}" + "${env.qa_registry}"
                        stage_registry_endpoint     = 'https://' + "${env.registryURI}" + "${env.stage_registry}"
                        qa_image                    = "${env.registryURI}" + "${env.qa_registry}" + ':' + "${env.COMMITID}"
                        stage_image                 = "${env.registryURI}" + "${env.stage_registry}" + ':' + "${env.COMMITID}"
                    }
            steps {
                script {
                    docker.withRegistry(qa_registry_endpoint, qa_dh_creds) {
                         docker.image(qa_image).pull()
                }
                sh 'echo Image pulled from QA'
                sh 'echo Taggig Docker image from QA to Stage'
                sh "docker tag ${env.qa_image} ${env.stage_image}"
                
                
                    docker.withRegistry(stage_registry_endpoint , stage_dh_creds) {
                            docker.image(env.stage_image).push()
                    }
                
                sh 'echo Image Pushed to STAGE'
                
                }
           }
            post {
                        always {
                            sh 'echo Cleaning docker Images from Jenkins.'
                            sh "docker rmi ${env.qa_image}"
                            sh "docker rmi ${env.stage_image}"
                        }
                    }
        }

        stage('Pull Tag push to Prod') {
            when {
                    expression {
                        params.account == 'prod'
                    }
                }

                environment {
                        stage_registry_endpoint = 'https://' + "${env.registryURI}" + "${env.stage_registry}"
                        prod_registry_endpoint  = 'https://' + "${env.registryURI}" + "${env.prod_registry}"
                        stage_image             = "${env.registryURI}" + "${env.stage_registry}" + ':' + "${env.COMMITID}"
                        prod_image              = "${env.registryURI}" + "${env.prod_registry}" + ':' + "${env.COMMITID}"
                    }
            steps {
                script {
                    docker.withRegistry(stage_registry_endpoint, stage_dh_creds) {
                        docker.image(stage_image).pull()
                            }

                
                sh 'echo Image pulled from QA'
                sh 'echo Taggig Docker image from stage to prod'
                sh "docker tag ${env.stage_image} ${env.prod_image}" 
                
                    docker.withRegistry(prod_registry_endpoint , prod_dh_creds) {
                            docker.image(env.prod_image).push()
                    }
                
                sh 'echo Image Pushed to Prod'
                
               }
            }
            post {
                        always {
                            sh 'echo Cleaning docker Images from Jenkins.'
                            sh "docker rmi ${env.stage_image}"
                            sh "docker rmi ${env.prod_image}"
                            echo 'Deleting Project now !! '
                            deleteDir()
                        }
                    }
        }

}
