pipeline {
  agent none
  environment {
	MAJOR_VERSION=1
  }
    stages {
      stage('Say Hello'){
         agent any
         steps {
	    sayHello 'Awesome Student!'
         }
      }
      stage('Unit Tests'){
     	agent { label 'master' }
	steps{
	  sh 'ant -f test.xml -v'
	  junit 'reports/result.xml'
	}
      }
      stage('build') {
	agent { label 'master' }
        steps {
          sh 'ant -f build.xml -v'
        }
        post {
           success {
             archiveArtifacts artifacts:  'dist/*.jar', fingerprint: true
           }  
         }
      }
      stage('deploy') {
	agent { label 'master' }
        steps {
           sh "mkdir -p /var/www/html/rectangles/all/${env.BRANCH_NAME}"
	   sh "cp dist/rectangle_${MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all/${env.BRANCH_NAME}"
        }
      }
      stage('Running on CentOS') {
        agent { label 'CentOS' }
        steps { 
           sh "wget http://pradeepthimmireddy4.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"     
	   sh "java -jar rectangle_${MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
        }  	
      }
      stage("Test on Debian"){
        agent { docker 'openjdk:8u121-jre' }
        steps { 
           sh "wget http://pradeepthimmireddy4.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"     
           sh "java -jar rectangle_${MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
        }     
      }

      stage("Promote to green"){
	agent { label 'master' }
        when {
	   branch 'master'
	}
	steps {
	   sh "mkdir -p /var/www/html/rectangles/all/${env.BRANCH_NAME}"
	   sh "cp /var/www/html/rectangles/all/${env.BRANCH_NAME}/rectangle_${MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/green"
        }
      }

      stage("Promote development branch to master"){
        agent { label 'master' }
        when {
           branch 'development'
        }
        steps {
	   echo "Stashing Any Local Changes"
	   sh "git stash"
	   echo "Checking out development Branch"
	   sh "git checkout development"
	   echo "Checking Out master Branch"
	   sh 'git pull origin'
	   sh "git checkout master"
	   echo "Merging Development into Master Branch"
	   sh 'git merge development'
	   echo "Pushing to Origin Master"
	   sh 'git push origin master'
	   echo "Tagging the relase"
	   sh "git tag rectangle-${MAJOR_VERSION}.${env.BUILD_NUMBER}"
	   sh "git push origin rectangle-${MAJOR_VERSION}.${env.BUILD_NUMBER}"
        }
	post {
        success {
          emailext(
            subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] Development Promoted to Master",
            body: """<p>'${env.JOB_NAME} [${env.BUILD_NUMBER}]' Development Promoted to Master":</p>
            <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
            to: "emandry@gmail.com"
          )
        }
      }
      } 
    }
    post {
    failure {
      emailext(
        subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] Failed!",
        body: """<p>'${env.JOB_NAME} [${env.BUILD_NUMBER}]' Failed!":</p>
        <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
        to: "emandry@gmail.com"
      )
    }
  }
}

