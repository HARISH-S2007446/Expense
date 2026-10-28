pipeline {
    agent any

    stages {

        stage('1. Checkout') {
            steps {
                checkout scm
                echo 'GitHub source code checked out successfully.'
            }
        }

        stage('2. Check Tools') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    $PYTHON = "C:\\Users\\hp\\expense_tracker\\.venv\\Scripts\\python.exe"
                    $DOCKER = "C:\\Users\\hp\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe"
                    $KUBECTL = "C:\\Users\\hp\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\kubectl.exe"

                    Write-Host "Checking Python..."
                    if (-not (Test-Path $PYTHON)) {
                        throw "Python not found: $PYTHON"
                    }
                    & $PYTHON --version
                    if ($LASTEXITCODE -ne 0) { throw "Python check failed" }

                    Write-Host "Checking Docker..."
                    if (-not (Test-Path $DOCKER)) {
                        throw "Docker not found: $DOCKER"
                    }
                    & $DOCKER --version
                    if ($LASTEXITCODE -ne 0) { throw "Docker check failed" }

                    Write-Host "Checking kubectl..."
                    if (-not (Test-Path $KUBECTL)) {
                        throw "kubectl not found: $KUBECTL"
                    }
                    & $KUBECTL version --client
                    if ($LASTEXITCODE -ne 0) { throw "kubectl check failed" }

                    Write-Host "All tools are available."
                '''
            }
        }

        stage('3. Install Dependencies') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"
                    $PYTHON = "C:\\Users\\hp\\expense_tracker\\.venv\\Scripts\\python.exe"

                    Write-Host "Installing Python dependencies..."

                    if (-not (Test-Path "requirements.txt")) {
                        throw "requirements.txt not found"
                    }

                    & $PYTHON -m pip install -r requirements.txt

                    if ($LASTEXITCODE -ne 0) {
                        throw "Dependency installation failed"
                    }

                    Write-Host "Dependencies installed successfully."
                '''
            }
        }

        stage('4. Test Application') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"
                    $PYTHON = "C:\\Users\\hp\\expense_tracker\\.venv\\Scripts\\python.exe"

                    Write-Host "Testing Flask application..."

                    & $PYTHON -c "from app import app; print('Flask application loaded successfully')"

                    if ($LASTEXITCODE -ne 0) {
                        throw "Flask application test failed"
                    }

                    Write-Host "Application test passed."
                '''
            }
        }

        stage('5. Build Docker Image') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"
                    $DOCKER = "C:\\Users\\hp\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe"

                    Write-Host "Building Expense Tracker V1 image..."

                    if (-not (Test-Path "Dockerfile")) {
                        throw "Dockerfile not found"
                    }

                    & $DOCKER build -t expense-tracker:v1 .

                    if ($LASTEXITCODE -ne 0) {
                        throw "Docker image build failed"
                    }

                    Write-Host "Verifying Docker image..."
                    & $DOCKER image inspect expense-tracker:v1

                    if ($LASTEXITCODE -ne 0) {
                        throw "Docker image verification failed"
                    }

                    Write-Host "Docker image built successfully."
                '''
            }
        }

        stage('6. Deploy to Kubernetes') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    $KUBECTL = "C:\\Users\\hp\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\kubectl.exe"
                    $env:KUBECONFIG = "C:\\Users\\hp\\.kube\\config"

                    Write-Host "Checking Kubernetes configuration..."

                    if (-not (Test-Path $env:KUBECONFIG)) {
                        throw "Kubernetes config file not found: $env:KUBECONFIG"
                    }

                    & $KUBECTL config get-contexts
                    if ($LASTEXITCODE -ne 0) {
                        throw "Unable to read Kubernetes contexts"
                    }

                    Write-Host "Selecting docker-desktop context..."
                    & $KUBECTL config use-context docker-desktop
                    if ($LASTEXITCODE -ne 0) {
                        throw "Could not select docker-desktop context"
                    }

                    Write-Host "Checking Kubernetes nodes..."
                    & $KUBECTL get nodes
                    if ($LASTEXITCODE -ne 0) {
                        throw "Cannot connect to Kubernetes cluster"
                    }

                    Write-Host "Checking deployment.yaml..."
                    if (-not (Test-Path "deployment.yaml")) {
                        throw "deployment.yaml not found in Jenkins workspace"
                    }

                    Write-Host "Applying Kubernetes configuration..."
                    & $KUBECTL apply -f deployment.yaml
                    if ($LASTEXITCODE -ne 0) {
                        throw "Failed to apply deployment.yaml"
                    }

                    Write-Host "Waiting for deployment rollout..."
                    & $KUBECTL rollout status deployment/expense-tracker --timeout=180s
                    if ($LASTEXITCODE -ne 0) {
                        throw "Deployment rollout failed"
                    }

                    Write-Host "Kubernetes deployment completed successfully."
                '''
            }
        }

        stage('7. Verify Deployment') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    $KUBECTL = "C:\\Users\\hp\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\kubectl.exe"
                    $env:KUBECONFIG = "C:\\Users\\hp\\.kube\\config"

                    Write-Host "Checking Kubernetes deployment..."
                    & $KUBECTL get deployments
                    if ($LASTEXITCODE -ne 0) {
                        throw "Failed to get deployments"
                    }

                    Write-Host "Checking pods..."
                    & $KUBECTL get pods
                    if ($LASTEXITCODE -ne 0) {
                        throw "Failed to get pods"
                    }

                    Write-Host "Checking services..."
                    & $KUBECTL get services
                    if ($LASTEXITCODE -ne 0) {
                        throw "Failed to get services"
                    }

                    Write-Host "Deployment verification completed."
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Expense Tracker pipeline completed.'
        }

        failure {
            echo 'FAILURE: Check Console Output for the failed stage.'
        }
    }
}