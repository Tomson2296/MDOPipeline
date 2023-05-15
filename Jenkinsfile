pipeline{
    agent any
    environment{
        DOCKERHUB_CREDENTIALS = credentials('tomson2296')
    }
    parameters{
        booleanParam(
            name: 'PROMOTE',
            defaultValue: false,
            description: "Czy chcesz wypromowac artefakt?"
        )
        string(
            name: 'VERSION',
            defaultValue: "",
            description: "Jaki numer wersji?"
        )
    }
    stages{
        stage('Prepare'){
            steps{
                echo "Preparing"
                sh "rm -rf MDOPipeline"
                sh "docker system prune --all --force"
                sh "git clone https://github.com/Tomson2296/MDOPipeline.git"
                sh "pwd"
                sh "ls -la"
              
                sh "touch buildLog.txt"
                sh "touch testLog.txt"
            }
        }
        stage('Build'){
            steps{
                echo "Building"
                sh "pwd"
                sh "ls -la"
               
                sh "docker build -t bldr . -f MDOPipeline/DockerfileBuild | tee buildLog.txt"
                sh "docker run --name bldr bldr"
            }
        }
        stage('Testing'){
             steps{
                echo "Testing"
                sh "pwd"
                sh "ls -la"
              
                sh "docker build -t tstr . -f MDOPipeline/DockerfileTest | tee testLog.txt"
                sh "docker run --name tstr tstr"
            }
        }
        stage('Deploy'){
            steps{
                echo "Deploying"
                sh "pwd"
                sh "ls -la"

                sh "docker rm bldr"
                sh "docker rm tstr"
                archiveArtifacts artifacts: "buildLog.txt"
                archiveArtifacts artifacts: "testLog.txt"
               
                script{
                    if(params.PROMOTE){
                        sh "mkdir ${params.VERSION}"
                        sh "cd ${params.VERSION}"
                        sh "docker run --name tstr tstr"
                    }
                    else{
                        sh 'echo "Failure"'
                    }
                }
            }
        }
        stage('Publish'){
            steps{
                echo "Publishing"
                script{
                    if(params.PROMOTE){
                        sh 'echo "Publishing image to DockerHub"'
                        sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                        
                        sh "docker tag tstr:latest tomson2296/tstr:${params.VERSION}"
                        sh "docker push tomson2296/tstr:${params.VERSION}"
                        sh "docker rm tstr"
                        
                        sh "cd ../"
                        sh "tar -czvf tstr-${params.VERSION}.tar.gz ${params.VERSION}/"
                        archiveArtifacts artifacts: "tstr-${params.VERSION}.tar.gz"
                    }
                    else{
                        sh 'echo "Failure"'        
                    }
                }
            }
        }
    }
}
