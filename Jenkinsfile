pipeline {
    agent none

    stages {
        stage('Debug') {
            agent any
            steps {
                echo "PARAMETERS"
                echo "Target architecture: ${params.conv_target_arch}"
                echo "Exec profile: ${params.execution_profile}"
                echo "Model link: ${params.model_download_link}"
                echo "pull model? ${params.pull_model}"
            }
        }
        stage('Pull the model from CometML') {
            agent { label 'linux-x86' }
            when {
                beforeAgent true;
                expression {
                    return params.pull_model
                }
            }
            steps {
                echo 'Pulling model from CometML'
                echo "Download link: ${params.model_download_link}"
                sh 'fallocate -l 1G original.model'

                stash includes: '*.model', name: 'model'
            }
        }
        stage('Pull the model from local') {
            agent { label 'linux-x86' }
            when {
                beforeAgent true;
                expression {
                    return !params.pull_model
                }
            }
            steps {
                echo 'Pulling model from cache'
                sh 'fallocate -l 1G original.model'

                stash includes: '*.model', name: 'model'
            }
        }
        stage('Converting the model') {
            agent { label 'linux-x86' }
            steps {
                echo 'Pulling data for quantization'
                sh '''
                    fallocate -l 1G largefile.txt
                    fallocate -l 1G evenlargerfile.txt
                '''
                echo 'Creating aux files'
                sh '''
                    touch aux_file.txt
                    touch input_list.txt
                    touch raw_inputs.txt                                   
                '''
                stash includes: '*.txt', name: 'data'

                unstash 'model'

                echo "Converting the model for target: ${params.conv_target_arch}"              
                sh "fallocate -l 1G converted-${params.conv_target_arch}.model"

                stash includes: '*.model', name: 'models'
            }
            post {
                always {
                    archiveArtifacts artifacts: '*.model', fingerprint: true
                }
            }
        }
        stage('Execute the model') {
            agent { label 'bench-01' }
            steps {
                echo 'Unstashing...'
                unstash 'data'
                sh 'ls -lah'
                unstash 'models'
                sh '''
                    ls -lah
                    echo "Executing the model"
                    echo "Done"
                    
                '''
            }
        }
        stage('Deploy') {
            agent { label 'linux-x86' }
            steps {
                echo 'Deploying....'
                // sh '''
                //     sleep 60
                // '''
            }
        }
    }
    
}
