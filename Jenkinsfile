def sfpath = "C:/ProgramData/jenkins/.jenkins/workspace/scm_dotnetapp/aspnet-core-dotnet-core/aspnet-core-dotnet-core.csproj"
def pfile = "aspnet-core-dotnet-core/bin/Debug/netcoreapp1.1/publish" 
def jfrogTargetPath = "D:/jfrog/"

pipeline 
{
    environment
    {
        appName = "rose-webapp"
        resourceGroup = "Training-rg"
        
    }
    agent any	
	stages 
	{
        stage('sonarqube') 
        {
            steps
            {
                script
                {
                    
                    def scannerHome = tool 'SonarScanner for MSBuild'
                    withSonarQubeEnv('SonarQubeServer') 
                    {
                        bat "\"${scannerHome}\\SonarScanner.MSBuild.exe\" begin /k:\"DotnetApp\""
                        
			                  bat "dotnet build ${sfpath}"    
                        bat "\"${scannerHome}\\SonarScanner.MSBuild.exe\" end"
                    }
                }
            }
        }
        stage("Quality gate") 
        {
            steps {
                waitForQualityGate abortPipeline: false
            }
        }
        stage('build')
        {
            steps
            {
                echo "Builds the project and its dependencies"
                bat "dotnet build ${sfpath}"
		    
            }
        }
        stage('test')
        {
            steps
            {
                bat "dotnet test ${sfpath}"
		    
            }
        }
        stage('publish')
        {
            steps
            {
                bat "dotnet publish ${sfpath}"
		    
            }
        }
        
        stage('Package') 
        {
            steps 
                {
                echo "Deploying to stage environment for more tests!";
                bat "del *.zip"
                
		bat "tar.exe -a -c -f WebApp_${BUILD_NUMBER}.zip ${pfile}"
                }
        }
        
        
        stage('file upload to jfrog')
        {
            steps{
                rtUpload (
                 serverId:"Artifactory" ,
                  spec: '''{
                   "files": [
                      {
                      "pattern": "${WORKSPACE}/WebApp_${BUILD_NUMBER}.zip",
                      "target": "Rose-dotnet-app"
                      }
                            ]
                           }''',
                        )
            }
        }
        stage ('Publish build info') 
        {
            steps 
            {
                rtPublishBuildInfo (
                    serverId: "Artifactory"
                )
            }
        }
stage ('download the artifacts from artifactory')
        {
            steps
            {
                  rtDownload (
                    serverId: "Artifactory",
                        spec:
                              """{
                                "files": [
                                  {
                                    "pattern": "Rose-dotnet-app/WebApp_${BUILD_NUMBER}.zip",
                                    "target": "${jfrogTargetPath}"          
                                  }
                               ]
                              }"""
      )
            }
        }
  
	stage('Deploy to azure') 
	{
	   steps
		{
		   
			azureWebAppPublish appName: "${env.appName}", azureCredentialsId: 'Azure', resourceGroup: "${env.resourceGroup}"
	    }
	}
	}
}


