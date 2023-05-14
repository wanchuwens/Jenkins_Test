node {
   def git_commit_id
   def to = emailextrecipients([
        [$class: 'CulpritsRecipientProvider'],
        [$class: 'DevelopersRecipientProvider'],
        [$class: 'RequesterRecipientProvider']
  ])
   
   
   stage('Prepare') {
     checkout scm
     sh "git rev-parse --short HEAD > .git/commit-id"
     git_commit_id = readFile('.git/commit-id').trim()
   }
   
   stage('test') {
     def testContainer = docker.image('node:4.6')
     testContainer.pull()
     testContainer.inside {
       sh 'npm install --only=dev'
       sh 'npm test'
     }
   }
   
   try {
   stage('Running Tests with a Database') {
     def mysql = docker.image('mysql').run("-e MYSQL_ALLOW_EMPTY_PASSWORD=yes") 
     def testContainer = docker.image('node:4.6')
     testContainer.pull()
     testContainer.inside("--link ${mysql.id}:mysql") {
          sh 'npm install --only=dev' 
          sh 'npm test'                     
     }     
     currentBuild.result = "SUCCESS";
     mysql.stop()
    }  
   } catch (Exception err) {
     currentBuild.result = "FAILURE";
     def subject = "${env.JOB_NAME} - Build #${env.BUILD_NUMBER} ${currentBuild.result}"
     def content = '${JELLY_SCRIPT,template="html"}'
     
     if(to != null && !to.isEmpty()) {
      emailext(body: content, mimeType: 'text/html',
         replyTo: '$DEFAULT_REPLYTO', subject: subject,
         to: to, attachLog: true )
    }
   }
   
   echo "RESULT_MYSQL_JOB: ${currentBuild.result}"

   
   stage('Pushing builds in Docker hub') {            
     docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
       def app = docker.build("grv231/nodejs-docker-jenkins-demo:${git_commit_id}", '.').push()
     }                                     
   }                                       
}                                          
