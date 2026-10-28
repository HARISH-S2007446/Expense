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

                Write-Host "Checking Python..."
                $PYTHON_CMD = Get-Command python.exe -ErrorAction SilentlyContinue

                if (-not $PYTHON_CMD) {
                    throw "Python is not available in Jenkins PATH."
                }

                $PYTHON_BASE = $PYTHON_CMD.Source
                & $PYTHON_BASE --version

                if ($LASTEXITCODE -ne 0) {
                    throw "Python check failed."
                }

                Write-Host "Checking Docker..."
                $DOCKER_CMD = Get-Command docker.exe -ErrorAction SilentlyContinue

                if (-not $DOCKER_CMD) {
                    throw "Docker is not available in Jenkins PATH."
                }

                $DOCKER = $DOCKER_CMD.Source
                & $DOCKER --version

                if ($LASTEXITCODE -ne 0) {
                    throw "Docker check failed."
                }

                Write-Host "Checking kubectl..."
                $KUBECTL_CMD = Get-Command kubectl.exe -ErrorAction SilentlyContinue

                if (-not $KUBECTL_CMD) {
                    throw "kubectl is not available in Jenkins PATH."
                }

                $KUBECTL = $KUBECTL_CMD.Source
                & $KUBECTL version --client

                if ($LASTEXITCODE -ne 0) {
                    throw "kubectl check failed."
                }

                Write-Host "All tools are available."
            '''
        }
    }

    stage('3. Install Dependencies') {
        steps {
            powershell '''
                $ErrorActionPreference = "Stop"

                if (-not (Test-Path "requirements.txt")) {
                    throw "requirements.txt not found."
                }

                $PYTHON_CMD = Get-Command python.exe -ErrorAction SilentlyContinue

                if (-not $PYTHON_CMD) {
                    throw "Python is not available in Jenkins PATH."
                }

                $PYTHON_BASE = $PYTHON_CMD.Source
                $VENV = Join-Path $env:WORKSPACE ".venv"
                $PYTHON = Join-Path $VENV "Scripts\python.exe"

                if (-not (Test-Path $PYTHON)) {
                    Write-Host "Creating virtual environment..."
                    & $PYTHON_BASE -m venv $VENV

                    if ($LASTEXITCODE -ne 0) {
                        throw "Virtual environment creation failed."
                    }
                }

                Write-Host "Installing dependencies..."
                & $PYTHON -m pip install --upgrade pip

                if ($LASTEXITCODE -ne 0) {
                    throw "Pip upgrade failed."
                }

                & $PYTHON -m pip install -r requirements.txt

                if ($LASTEXITCODE -ne 0) {
                    throw "Dependency installation failed."
                }

                Write-Host "Dependencies installed successfully."
            '''
        }
    }

    stage('4. Test Application') {
        steps {
            powershell '''
                $ErrorActionPreference = "Stop"

                $PYTHON = Join-Path $env:WORKSPACE ".venv\Scripts\python.exe"

                if (-not (Test-Path $PYTHON)) {
                    throw "Python virtual environment not found."
                }

                if (-not (Test-Path "app.py")) {
                    throw "app.py not found."
                }

                Write-Host "Testing Flask application..."
                & $PYTHON -c "from app import app; print('Flask application loaded successfully')"

                if ($LASTEXITCODE -ne 0) {
                    throw "Application test failed."
                }

                Write-Host "Application test passed."
            '''
        }
    }

    stage('5. Build Docker Image') {
        steps {
            powershell '''
                $ErrorActionPreference = "Stop"

                $DOCKER_CMD = Get-Command docker.exe -ErrorAction SilentlyContinue

                if (-not $DOCKER_CMD) {
                    throw "Docker is not available in Jenkins PATH."
                }

                $DOCKER = $DOCKER_CMD.Source

                if (-not (Test-Path "Dockerfile")) {
                    throw "Dockerfile not found."
                }

                Write-Host "Checking Docker connection..."
                & $DOCKER info

                if ($LASTEXITCODE -ne 0) {
                    throw "Docker is not running or Jenkins cannot access Docker."
                }

                Write-Host "Building Expense Tracker image..."
                & $DOCKER build -t expense-tracker:v1 .

                if ($LASTEXITCODE -ne 0) {
                    throw "Docker image build failed."
                }

                & $DOCKER image inspect expense-tracker:v1

                if ($LASTEXITCODE -ne 0) {
                    throw "Docker image verification failed."
                }

                Write-Host "Docker image built successfully."
            '''
        }
    }

    stage('6. Deploy to Kubernetes') {
        steps {
            powershell '''
                $ErrorActionPreference = "Stop"

                $KUBECTL_CMD = Get-Command kubectl.exe -ErrorAction SilentlyContinue

                if (-not $KUBECTL_CMD) {
                    throw "kubectl is not available in Jenkins PATH."
                }

                $KUBECTL = $KUBECTL_CMD.Source

                $KUBECONFIG_PATH = Join-Path $env:USERPROFILE ".kube\config"

                if (-not (Test-Path $KUBECONFIG_PATH)) {
                    throw "Kubernetes config not found: $KUBECONFIG_PATH"
                }

                $env:KUBECONFIG = $KUBECONFIG_PATH

                Write-Host "Selecting Kubernetes context..."
                & $KUBECTL config use-context docker-desktop

                if ($LASTEXITCODE -ne 0) {
                    throw "Could not select docker-desktop context."
                }

                Write-Host "Checking Kubernetes nodes..."
                & $KUBECTL get nodes

                if ($LASTEXITCODE -ne 0) {
                    throw "Cannot connect to Kubernetes cluster."
                }

                if (-not (Test-Path "deployment.yaml")) {
                    throw "deployment.yaml not found."
                }

                Write-Host "Applying Kubernetes deployment..."
                & $KUBECTL apply -f deployment.yaml

                if ($LASTEXITCODE -ne 0) {
                    throw "Kubernetes deployment failed."
                }

                Write-Host "Waiting for deployment rollout..."
                & $KUBECTL rollout status deployment/expense-tracker --timeout=180s

                if ($LASTEXITCODE -ne 0) {
                    throw "Deployment rollout failed."
                }

                Write-Host "Kubernetes deployment completed successfully."
            '''
        }
    }

    stage('7. Verify Deployment') {
        steps {
            powershell '''
                $ErrorActionPreference = "Stop"

                $KUBECTL_CMD = Get-Command kubectl.exe -ErrorAction SilentlyContinue

                if (-not $KUBECTL_CMD) {
                    throw "kubectl is not available in Jenkins PATH."
                }

                $KUBECTL = $KUBECTL_CMD.Source
                $KUBECONFIG_PATH = Join-Path $env:USERPROFILE ".kube\config"

                if (-not (Test-Path $KUBECONFIG_PATH)) {
                    throw "Kubernetes config not found."
                }

                $env:KUBECONFIG = $KUBECONFIG_PATH

                Write-Host "Checking deployments..."
                & $KUBECTL get deployments

                if ($LASTEXITCODE -ne 0) {
                    throw "Failed to get deployments."
                }

                Write-Host "Checking pods..."
                & $KUBECTL get pods

                if ($LASTEXITCODE -ne 0) {
                    throw "Failed to get pods."
                }

                Write-Host "Checking services..."
                & $KUBECTL get services

                if ($LASTEXITCODE -ne 0) {
                    throw "Failed to get services."
                }

                Write-Host "Deployment verification completed successfully."
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
