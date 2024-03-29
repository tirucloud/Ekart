pipeline {
  agent any
  tools {
    maven 'maven3'
    jdk 'jdk17'
  }
  environment {
    SCANNER_HOME = tool 'sonar-scanner'
  }
  stages {
    stage('Git Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/tirucloud/Ekart.git'
      }
    }
    stage('Compile') {
      steps {
        sh "mvn compile"
      }
    }
    stage('Unit Tests') {
      steps {
        sh "mvn test -DskipTests=true"
      }
    }
    stage('SonarQube Analysis') {
     steps {
    withSonarQubeEnv('sonar-server') {
      sh '''
        $SCANNER_HOME/bin/sonar-scanner \
        -Dsonar.projectKey=EKART \
        -Dsonar.projectName=EKART \
        -Dsonar.java.binaries=.
      '''
    }
  }
}
    stage('OWASP Dependency Check') {
      steps {
        dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DC'
        dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
      }
    }
    
    stage('Build') {
      steps {
        sh "mvn package -DskipTests=true"
      }
    }
   
    stage('Docker Build & tag Image') {
      steps {
        script {
          withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
            sh "docker build -t tirucloud/ekart:latest -f docker/Dockerfile ."
          }
        }
      }
    }
     stage('Trivy Scan') {
      steps {
        sh "trivy image tirucloud/ekart:latest > trivy-report.txt "
      }
    }
    stage('Docker Push Image') {
      steps {
        script {
          withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
            sh "docker push tirucloud/ekart:latest"
          }
        }
      }
    }
    stage('Kubernetes Deploy') {
      steps {
        withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.8.162:6443') {
          sh "kubectl apply -f deploymentservice.yml -n webapps"
          sh "kubectl get svc -n webapps"
        }
      }
    }
    
  }
}
