pipeline {
    agent any
    stages {
        stage('Git Checkout') {
            steps {
                script {
                    git branch: 'main',
                        credentialsId: 'Credential ID',
                        url: 'https://github.com/deepak-323/Jenkins.git'
                }
            }
        }
	stage('Install dependencies') {
            steps {
                // Install Python dependencies
                sh 'pip install -r requirements.txt'
            }
        }
	stage('Test') {
            steps {
                script { // Your build commands go here 
	          sh "chmod +x -R ${env.WORKSPACE}" 
		  sh '/usr/bin/python3 /var/lib/jenkins/workspace/pipeline-1/TF_Inference_cifar.py'
		  sh "echo ${currentBuild.number}"
	       }
            }
        }
	stage('Archive Artifacts') {
            steps {
                // Archive the artifacts (output files) generated by your script
                archiveArtifacts artifacts: '*'
            }
        }
	stage('Create Tar File') {
            steps {
                script {
                    def buildNumber = env.BUILD_NUMBER
                    def outputDir = "/var/lib/jenkins/jobs/pipeline-1/builds/${currentBuild.number}"

                    // Check if the directory exists
                    if (fileExists(outputDir)) {
                        // Create a tar file
                        sh "sleep 5 && tar --warning=no-file-changed -czvf /var/lib/jenkins/workspace/pipeline-1/output.tar.gz --exclude='*.tmp' --ignore-failed-read -C ${outputDir} ."
                    } else {
                        error "Output directory not found: ${outputDir}"
                   }
                }
            }
        }
	stage('Upload to S3') {
            steps {
                script {
                    def outputDir = "/var/lib/jenkins/workspace/pipeline-1"
		    sh "aws s3 cp ${outputDir}/output.tar.gz s3://mcw-pipeline/Artifacts/pipeline-1/${currentBuild.number}/"

                }
            }
        }
    }
    environment {
        BUILD_URL = "http://15.206.159.119:8080/job/pipeline-1/${currentBuild.number}"
    }
    post {
      failure {
            emailext subject: "Failed: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                      body: "Build URL: ${env.BUILD_URL}",
                      to: 'deepak.sudhakar@multicorewareinc.com',
                      replyTo: 'deepak.sudhakar@multicorewareinc.com',
                      mimeType: 'text/html'
        }
        success {
            emailext subject: "Success: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                      body: "Build URL: ${env.BUILD_URL}",
                      to: 'deepak.sudhakar@multicorewareinc.com',
                      replyTo: 'deepak.sudhakar@multicorewareinc.com',
                      mimeType: 'text/html'
        }
}
}

