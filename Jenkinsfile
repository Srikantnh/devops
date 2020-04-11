pipeline {
   agent any
   environment {
       registry = "global.samsung.repo/csr-portal"        
       CSR-APP = "/opt/CSR-Pro" 
   }
   stages {
       stage('Build') {
           agent {
               docker {
                   image 'csr:2.5'
				   WORKDIR 'CSR-App'
               }
           }
           steps {
               // Create our project directory.
               sh 'cd ${CSR-APP}/src'
               sh 'mkdir -p ${CSR-APP}/src/csr-1-app'
               // Copy all files in our Jenkins workspace to our project directory.
               sh 'cp -r ${WORKSPACE}/* ${CSR-APP}/src/csr-1-app'
               // Build the app.
               sh 'mvn install'
           }
       }
       stage('Test') {
           agent {
               docker {
                   image 'golang'
               }
           }
           steps {
               // Create our project directory.
               sh 'cd ${CSR-APP}/src'
               sh 'mkdir -p ${CSR-APP}/src/csr-1-app'
               // Copy all files in our Jenkins workspace to our project directory.
               sh 'cp -r ${WORKSPACE}/* ${CSR-APP}/src/csr-1-app'
               // Remove cached test results.
               sh 'mvn clean -cache'
               // Run Unit Tests.
               sh 'mvn test ./... -v -short'
           }
       }
       stage('Publish') {
           environment {
               registryCredential = 'dockerhub'
           }
           steps{
               script {
                   def appimage = docker.build registry + ":$BUILD_NUMBER"
                   docker.withRegistry( '', registryCredential ) {
                       appimage.push()
                       appimage.push('latest')
                   }
               }
           }
       }
       stage ('Deploy') {
           steps {
               script{
                   def image_id = registry + ":$BUILD_NUMBER"
                   sh "ansible-playbook  playbook.yml --extra-vars \"image_id=${image_id}\""
               }
           }
       }
   }
}
