pipeline {
  agent any

  environment {
    AWS_REGION = 'eu-north-1'
    CLUSTER_NAME = 'my-eks-cluster'
  }

  stages {
    stage('Checkout Terraform Code') {
      steps {
        git 'https://your-repo-url/terraform-eks.git'
      }
    }

    stage('Terraform Init & Apply') {
      steps {
        sh 'terraform init'
        sh 'terraform apply -auto-approve'
      }
    }

    stage('Configure kubectl') {
      steps {
        sh '''
          aws eks --region $AWS_REGION update-kubeconfig --name $CLUSTER_NAME
          kubectl get nodes
        '''
      }
    }

    stage('Install Gatekeeper with Helm') {
      steps {
        sh '''
          helm repo add gatekeeper https://open-policy-agent.github.io/gatekeeper/charts
          helm repo update
          helm install gatekeeper gatekeeper/gatekeeper --namespace gatekeeper-system --create-namespace
          kubectl rollout status deployment gatekeeper-controller-manager -n gatekeeper-system
        '''
      }
    }

    stage('Apply ConstraintTemplates & Constraints') {
      steps {
        sh '''
          kubectl apply -f templates/allowed-namespaces-template.yaml
          kubectl apply -f constraints/allowed-namespaces-constraint.yaml

          kubectl apply -f templates/k8sdenylatest-template.yaml
          kubectl apply -f constraints/deny-latest-constraint.yaml

          kubectl apply -f templates/nonroot-template.yaml
          kubectl apply -f constraints/nonroot-constraint.yaml

          kubectl apply -f templates/privileged-container-template.yaml
          kubectl apply -f constraints/privileged-container-constraint.yaml
        '''
      }
    }

    stage('Deploy and Test Bad Pod') {
      steps {
        script {
          def output = sh(script: 'kubectl apply -f pods/bad-pod.yaml || true', returnStdout: true).trim()
          echo "\u001B[31mBad Pod Deployment Result:\u001B[0m\n${output}"
        }
      }
    }

    stage('Verify Gatekeeper Violations') {
      steps {
        sh '''
          echo "\n[Gatekeeper Violations]"
          kubectl get constraints --all-namespaces
          kubectl describe constrainttemplates
        '''
      }
    }
  }

  post {
    always {
      echo 'Pipeline completed.'
    }
  }
}
