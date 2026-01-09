pipeline
{
	agent any
	environment
	{
		IMAGE_NAME="karthirs/minikubedemo"
		APP_NAME="minidemo"
		CONTAINER_NAME="my-maven-container"
	}

	stages
	{	
		stage('Checkout')
		{
			steps
			{
				git url:'git@github.com:BKarthik03/Maven_Kube_Demo.git', branch:'main'
			}
		}

		stage('Build Maven Project')
		{
			steps
			{
				sh 'mvn clean package -DskipTests'
			}
		}

		stage('Build Docker Image')
		{
			steps
			{
				script
				{
					dockerImage=docker.build("${IMAGE_NAME}:latest")
				}
			}
		}

		stage('Push DockerImage into HUB')
		{
			steps
			{
				script
				{
					docker.withRegistry('https://index.docker.io/v1/','dockerhub')
					{
						dockerImage.push()
					}
				}
			}
		}

		stage('Deploy in MiniKube')
		{
			steps
			{
				script
				{
					sh '''
					echo "Checking if Deployment Exist"
				
					if kubectl get deployment ${APP_NAME} >/dev/null 2>&1; then
						echo "Deployment Exists"
						kubectl set image deployment/${APP_NAME} \
						${CONTAINER_NAME}=${IMAGE_NAME}:latest
					else
						echo "Creating Deployment.."
						kubectl create deployment ${APP_NAME}\
						--image=${IMAGE_NAME}:latest

						kubectl expose deployment ${APP_NAME}\
						--type=NodePort \
						--port=8080
					fi
					'''

				}
			}
		}
	}

	post
	{
		success
		{
			echo "CI/CD Pipeline Successful!"
		}	
		failure
		{
			echo "Pipeline Failed!"
		}
		always
		{
			echo "Cleaning Workspace.."
			deleteDir()
		}
	}
}
