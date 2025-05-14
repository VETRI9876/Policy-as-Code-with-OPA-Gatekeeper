pipeline {
  agent any

  environment {
    AWS_REGION = 'eu-north-1'
    CLUSTER_NAME = 'my-eks-cluster'
  }

  stages {
    stage('Checkout Terraform Code') {
      steps {
        bat """
          git clone --branch main https://github.com/VETRI9876/Policy-as-Code-with-OPA-Gatekeeper.git
        """
      }
    }

    stage('Configure kubectl') {
      steps {
        bat """
          echo Running kubectl get nodes...
          kubectl get nodes
        """
      }
    }

    stage('Install Gatekeeper with Helm') {
      steps {
        bat """
          echo Adding Gatekeeper Helm repository...
          helm repo add gatekeeper https://open-policy-agent.github.io/gatekeeper/charts
          helm repo update
          echo Installing Gatekeeper...
          helm install gatekeeper gatekeeper/gatekeeper --namespace gatekeeper-system --create-namespace
          echo Waiting for Gatekeeper rollout...
          kubectl rollout status deployment gatekeeper-controller-manager -n gatekeeper-system
        """
      }
    }

    stage('Apply ConstraintTemplates & Constraints') {
      steps {
        bat """
          echo Applying allowed-namespaces-template.yaml...
          kubectl apply -f allowed-namespaces-template.yaml
          kubectl get constraints --all-namespaces

          echo Applying allowed-namespaces-constraint.yaml...
          kubectl apply -f allowed-namespaces-constraint.yaml
          kubectl get constraints --all-namespaces

          echo Applying k8sdenylatest-template.yaml...
          kubectl apply -f k8sdenylatest-template.yaml
          kubectl get constraints --all-namespaces

          echo Applying deny-latest-constraint.yaml...
          kubectl apply -f deny-latest-constraint.yaml
          kubectl get constraints --all-namespaces

          echo Applying nonroot-template.yaml...
          kubectl apply -f nonroot-template.yaml
          kubectl get constraints --all-namespaces

          echo Applying nonroot-constraint.yaml...
          kubectl apply -f nonroot-constraint.yaml
          kubectl get constraints --all-namespaces

          echo Applying privileged-container-template.yaml...
          kubectl apply -f privileged-container-template.yaml
          kubectl get constraints --all-namespaces

          echo Applying privileged-container-constraint.yaml...
          kubectl apply -f privileged-container-constraint.yaml
          kubectl get constraints --all-namespaces
        """
      }
    }

    stage('Deploy and Test Bad Pod') {
      steps {
        bat """
          echo Deploying bad pod...
          kubectl apply -f bad-pod.yaml
          echo Checking pod status...
          kubectl get pods
        """
      }
    }

    stage('Verify Gatekeeper Violations') {
      steps {
        bat """
          echo [Gatekeeper Violations]...
          kubectl get constraints --all-namespaces
          kubectl describe constrainttemplates
        """
      }
    }
  }

  post {
    always {
      echo 'Pipeline completed.'
    }
    success {
      echo 'Pipeline succeeded!'
    }
    failure {
      echo 'Pipeline failed!'
    }
  }
}
