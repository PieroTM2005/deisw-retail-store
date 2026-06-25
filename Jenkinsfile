pipeline {
  agent any
  tools {
    maven 'MAVEN_3_9_16'
    jdk 'JDK_26'
  }
	environment {
		    REGISTRY_USER = "pierotm" 
        // Nombre de la imagen que vamos a crear para nuestra aplicación
        IMAGE_NAME = "retail-store-u202318731"
        TAG        = "${env.BUILD_NUMBER}" // Usa el número de ejecución de Jenkins como versión
    }

  stages {
    stage ('Compile Project') {
      steps {
        withMaven(maven : 'MAVEN_3_9_16') {
            sh 'mvn clean compile'
        }
      }
    }

    stage('Validate Checkstyle') {
      steps {
        withMaven(maven: 'MAVEN_3_9_16') {
          sh 'mvn checkstyle:check'
        }
      }
    }

    stage('Validate Unit Tests') {
      steps {
        withMaven(maven: 'MAVEN_3_9_16') {
          sh 'mvn test'
        }
      }
    }

    stage('Validate Test Coverage') {
      steps {
        withMaven(maven: 'MAVEN_3_9_16') {
          sh 'mvn clean verify jacoco:report'
          sh 'mvn jacoco:check'
        }
      }
    }

	 stage ('SonarQube Analysis') {
        steps {
			// 1. Enviar el código a analizar a SonarQube
            withSonarQubeEnv('MiSonarServer') {
                sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=learning-center'
            }
			// 2. Pausar el pipeline y esperar la respuesta del Webhook de SonarQube
	        script {
	            timeout(time: 10, unit: 'MINUTES') { // Evita que se quede bloqueado permanentemente si cae la red
	                // Este paso intercepta la notificación enviada al puerto 9089
	                def qg = waitForQualityGate()
	                
	                // 3. Evaluar el estado del Quality Gate
	                if (qg.status != 'OK') {
	                    error "El pipeline se ha detenido porque el código no superó el Quality Gate de SonarQube. Estado: ${qg.status}"
	                }
	            }
	        }

        }
     }
	  stage('Construir y Publicar Imagen Docker') {
            steps {
                // Nos autenticamos de forma segura en Docker Hub usando el ID de credenciales de Jenkins
                withCredentials([usernamePassword(credentialsId: 'DOCKER_HUB_CREDENTIALS', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        echo "Iniciando sesión en Docker Hub..."
                        sh "echo '${DOCKER_PASS}' | docker login -u '${DOCKER_USER}' --password-stdin"

                        echo "Construyendo imagen optimizada AMD64..."
                        sh "docker buildx build --platform linux/amd64 -t $pierotm/retail-store-u202318731:${TAG} -t pierotm/retail-store-u202318731:latest --push ."
                    }
                }
            }
        }
	  


    }
}
