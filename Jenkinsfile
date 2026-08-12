<![CDATA[
// Source: https://github.com/jenkinsci/pipeline-examples (MIT License)  
// Test case: Node.js build with Docker and deployment
node('node') {
    currentBuild.result = "SUCCESS"

    try {
       stage('Checkout'){
          checkout scm
       }

       stage('Install Dependencies'){
         env.NODE_ENV = "development"
         sh 'node -v'
         sh 'npm -v'
         sh 'npm install'
       }

       stage('Test'){
         env.NODE_ENV = "test"
         print "Environment will be : ${env.NODE_ENV}"
         sh 'npm test'
         publishTestResults testResultsPattern: 'test-results.xml'
       }

       stage('Build'){
         env.NODE_ENV = "production"
         sh 'npm run build'
         archiveArtifacts artifacts: 'dist/**/*', fingerprint: true
       }

       stage('Build Docker'){
         sh 'docker build -t myapp:${BUILD_NUMBER} .'
         sh 'docker tag myapp:${BUILD_NUMBER} myapp:latest'
       }

       stage('Deploy'){
         echo 'Push to Registry'
         sh 'docker push myapp:${BUILD_NUMBER}'
         sh 'docker push myapp:latest'
         
         echo 'Deploy to server'
         sh 'kubectl set image deployment/myapp myapp=myapp:${BUILD_NUMBER}'
       }

       stage('Cleanup'){
         sh 'npm prune --production'
         sh 'docker system prune -f'
       }

    } catch (err) {
        currentBuild.result = "FAILURE"
        throw err
    }
}
    ]]>