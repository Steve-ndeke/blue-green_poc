pipeline {
    agent any
    environment {
        K8S_NAMESPACE = 'green'
        HELM_RELEASE_NAME = 'blue-green-deployment'
        CHART_PATH = './blue-green-deployment/charts'
        KUBECTL_PATH = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe'
        HELM_VERSION = 'v3.15.4'
        HELM_INSTALL_DIR = "${env.WORKSPACE}/helm"
        PATH = "${HELM_INSTALL_DIR}:${env.PATH}"
        KUBECTL_INSTALL_DIR = "$HOME/bin"
    }
    stages {
        stage('Check and Install kubectl') {
            steps {
                script {
                    // Use 'where' instead of 'which' for Windows
                    def kubectlExists = bat(script: 'where kubectl', returnStatus: true) == 0

                    if (kubectlExists) {
                        echo "kubectl is already installed."
                    } else {
                        echo "kubectl not found. Installing..."
                        // Ensure the directory path is correctly specified
                        def kubectlDir = "${env.WORKSPACE}\\kubectl"
                        bat """
                        mkdir "${kubectlDir}"
                        curl -LO "https://dl.k8s.io/release/v1.25.0/bin/windows/amd64/kubectl.exe"
                        move kubectl.exe "${kubectlDir}\\kubectl.exe"
                        """
                        // Add the directory to PATH
                        bat """
                        setx PATH "%PATH%;${kubectlDir}"
                        """
                        echo "kubectl installed successfully."
                    }
                }
            }
        }


        stage('Test Kubernetes Access') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    bat '"%KUBECTL_PATH%" get nodes --kubeconfig %KUBECONFIG%'
                }
            }
        }

        stage('Install Helm') {
            steps {
                script {
                    // Check if Helm is installed
                    def helmExists = bat(script: 'where helm', returnStatus: true) == 0

                    if (helmExists) {
                        echo "Helm is already installed."
                    } else {
                        echo "Helm is not installed. Installing Helm..."
                        bat """
                        mkdir %HELM_INSTALL_DIR%
                        curl -L https://get.helm.sh/helm-${HELM_VERSION}-windows-amd64.zip -o helm.zip
                        powershell -command "Expand-Archive helm.zip -DestinationPath ${HELM_INSTALL_DIR}"
                        move ${HELM_INSTALL_DIR}\\windows-amd64\\helm.exe ${HELM_INSTALL_DIR}\\helm.exe
                        del helm.zip
                        """
                    }
                }
            }
        }

        stage('Helm Upgrade or Install') {
            steps {
                script {
                    helmUpgradeOrInstall()
                }
            }
        }
    }
}

// Function definition outside the pipeline block
def helmUpgradeOrInstall() {
    def deploymentExists = bat(script: "helm list -n ${env.K8S_NAMESPACE} | findstr ${env.HELM_RELEASE_NAME}", returnStatus: true) == 0

    if (deploymentExists) {
        echo "Deployment exists. Upgrading..."
        echo "Executing: helm upgrade ${env.HELM_RELEASE_NAME} ${env.CHART_PATH} -n ${env.K8S_NAMESPACE}"
        bat "helm upgrade ${env.HELM_RELEASE_NAME} ${env.CHART_PATH} -n ${env.K8S_NAMESPACE}"
    } else {
        echo "Deployment does not exist. Installing..."
        echo "Executing: helm install ${env.HELM_RELEASE_NAME} ${env.CHART_PATH} -n ${env.K8S_NAMESPACE} --create-namespace"
        bat "helm install ${env.HELM_RELEASE_NAME} ${env.CHART_PATH} -n ${env.K8S_NAMESPACE} --create-namespace"
    }
}

