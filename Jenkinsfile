pipeline
{
   agent any
   tools
	{
		maven 'Maven 3.6.3'
	}
   options
   {
      timeout(time: 1, unit: 'HOURS')
      buildDiscarder(logRotator(daysToKeepStr: '5', numToKeepStr: '5'))
      disableConcurrentBuilds()
   }
    stages
    {
	        stage ('checkout')
		{
		    steps
		    {
			checkout scm
		    }
		}
		stage ('Build')
		{
		    steps
		    {
			bat 'mvn --version'
			bat 'mvn --install'
		    }
		}
		stage ('Unit Testing')
		{
		     steps
		     {
			 bat 'mvn --version'
			 bat 'mvn test'
		     }
		}
		stage ('Sonar Analysis')
		{
		     steps
		     {
		    	withSonarQubeEnv('Test_Sonar')
			{
    			    bat 'mvn --version'
    			    bat 'mvn sonar:sonar'
			}
		      }
		}
		stage ('Upload to Artifactory')
		{
		    steps
		    {
			rtMavenDeployer (
                                        id: 'deployer',
                    			serverId: 'art-1',
                    			releaseRepo: 'hariom_nagp',
                    			snapshotRepo: 'hariom_nagp'
                			)
                	rtMavenRun (
                    		    pom: 'pom.xml',
                    		    goals: 'clean install',
                    		    deployerId: 'deployer',
                                    )
			rtPublishBuildInfo (
			   	    	    serverId: 'art-1',
			            	   )
		    }
		}
		stage ('Docker Image')
		{
		    steps
		    {
			sh returnStdout: true, script: '/bin/docker build -t dtr.nagarro.com:443/devopssampleapplication_hariom:${BUILD_NUMBER} -f Dockerfile .'
		     }
		}
		stage ('Push to DTR')
	        {
		    steps
		    {
		    	sh returnStdout: true, script: '/bin/docker push dtr.nagarro.com:443/devopssampleapplication_hariom:${BUILD_NUMBER}'
		    }
	         }
       		stage ('Stop Running container')
    		{
	             steps
	        	 {
			    sh '''
			    ContainerID=$(docker ps | grep 7033 | cut -d " " -f 1)
			    if [  $ContainerID ]
			    then
				docker stop $ContainerID
				docker rm -f $ContainerID
			    fi
			    '''
	        	}
	        }
		stage ('Docker deployment')
		{
		    steps
		    {
		        sh 'docker run --name devopssampleapplication_hariom -d -p 7033:8080 dtr.nagarro.com:443/devopssampleapplication_hariom:${BUILD_NUMBER}'
		    }
		}
    }
}
