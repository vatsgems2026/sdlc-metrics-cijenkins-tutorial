pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: build
    image: ubuntu:24.04
    command:
    - sleep
    args:
    - 99d
  - name: trivy
    image: aquasec/trivy:0.58.0
    command:
    - cat
    tty: true
  - name: sonarqube
    image: sonarsource/sonar-scanner-cli:latest
    command:
    - cat
    tty: true
'''
        }
    }

    environment {
        // Application metadata
        APP_NAME = "sdlc-demo-app"

        // Build directories
        BUILD_DIR = "build"
        TEST_RESULTS_DIR = "test-results"
        DIST_DIR = "dist"

        // SonarQube credentials (for SAST scanning - optional)
        // TODO: Uncomment the lines below if you want to add SonarQube SAST scanning
        // SONAR_HOST = credentials('sonarqube-url')
        // SONAR_TOKEN = credentials('sonarqube-token')
    }

    stages {
        stage('Setup Environment') {
            steps {
                container('build') {
                    echo "Installing build tools..."
                    sh '''
                        apt-get update && apt-get install -y \
                            build-essential \
                            python3 \
                            python3-pip \
                            git \
                            tar

                        # Configure git to trust the workspace directory
                        git config --global --add safe.directory ${WORKSPACE}

                        # Create necessary directories
                        mkdir -p ${BUILD_DIR}
                        mkdir -p ${TEST_RESULTS_DIR}

                        # Display tool versions
                        echo "=== Build Tool Versions ==="
                        gcc --version | head -1
                        python3 --version
                        pip3 --version
                        make --version | head -1
                    '''
                }
            }
        }

        stage('Initialize') {
            steps {
                container('build') {
                    script {
                        echo "=== SDLC Metrics Jenkins Demo Pipeline ==="
                        echo "Build Number: ${env.BUILD_NUMBER}"
                        echo "Branch: ${env.BRANCH_NAME}"

                        // Capture Git commit info
                        env.GIT_COMMIT_HASH = sh(
                            script: 'git rev-parse HEAD',
                            returnStdout: true
                        ).trim()

                        env.GIT_COMMIT_SHORT = sh(
                            script: 'git rev-parse --short HEAD',
                            returnStdout: true
                        ).trim()

                        // Generate version string
                        def commitCount = sh(
                            script: 'git rev-list --count HEAD',
                            returnStdout: true
                        ).trim()
                        env.APP_VERSION = "1.0.${commitCount}-${env.GIT_COMMIT_SHORT}"

                        echo "Git Commit: ${env.GIT_COMMIT_SHORT}"
                        echo "Version: ${env.APP_VERSION}"

                        // Display repository info
                        sh '''
                            echo "Repository: $(git config --get remote.origin.url)"
                            echo "Branch: $(git rev-parse --abbrev-ref HEAD)"
                            echo "Last Commit: $(git log -1 --pretty=format:'%h - %s (%an)')"
                        '''
                    }
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                container('build') {
                    echo "Installing Python dependencies..."
                    sh '''
                        pip3 install --break-system-packages -r requirements.txt
                    '''
                }
            }
        }

        stage('Build and Package Application') {
            steps {
                container('build') {
                    echo "Building C application..."
                    sh '''
                        make clean
                        make all

                        # Verify build artifacts
                        ls -lh build/

                        echo "C application built successfully"
                    '''

                    echo "Building Python application..."
                    sh '''
                        # Compile Python files to check for syntax errors
                        python3 -m py_compile src/python/app.py

                        echo "Python application validated successfully"
                    '''

                    echo "Packaging application artifacts..."
                    sh """
                        # Create distribution directory
                        mkdir -p ${DIST_DIR}

                        # Copy C application binary
                        cp ${BUILD_DIR}/calculator ${DIST_DIR}/

                        # Copy Python application
                        mkdir -p ${DIST_DIR}/python-app
                        cp -r src/python/* ${DIST_DIR}/python-app/
                        cp requirements.txt ${DIST_DIR}/python-app/

                        # Copy documentation
                        cp README.md ${DIST_DIR}/ || echo "README.md not found, skipping"

                        # Create tarball
                        tar -czf ${APP_NAME}-${APP_VERSION}.tar.gz -C ${DIST_DIR} .

                        echo "=== Package Created ==="
                        echo "File: ${APP_NAME}-${APP_VERSION}.tar.gz"
                        ls -lh ${APP_NAME}-${APP_VERSION}.tar.gz

                        # Calculate checksum
                        sha256sum ${APP_NAME}-${APP_VERSION}.tar.gz
                    """

                    // Archive the artifact in Jenkins
                    archiveArtifacts artifacts: "${APP_NAME}-${APP_VERSION}.tar.gz", fingerprint: true
                }
            }
            // TODO: Uncomment the post block below to register the artifact with CloudBees Unify
             post {
                 success {
                     script {
                         // Calculate artifact checksum for registration
                         def artifactDigest = sh(
                             script: "sha256sum ${APP_NAME}-${APP_VERSION}.tar.gz | awk '{print \$1}'",
                             returnStdout: true
                         ).trim()
            
                         // Register build artifact with CloudBees Unify
                         // IMPORTANT: Capture the return value to get artifact ID for deployment tracking
                         def buildArtifactId = registerBuildArtifactMetadata(
                             name: "${APP_NAME}",
                             url: "${BUILD_URL}artifact/${APP_NAME}-${APP_VERSION}.tar.gz",
                             version: "${APP_VERSION}",
                             type: "Binary",
                             digest: artifactDigest,
                             label: "build-${BUILD_NUMBER},${BRANCH_NAME}"
                         )
            
                         // Store artifact ID for deployment stage
                         env.ARTIFACT_ID = buildArtifactId
                         echo "Build artifact registered with CloudBees Unify"
                         echo "Artifact ID: ${env.ARTIFACT_ID}"
                     }
                 }
             }
        }

        stage('Run C Unit Tests') {
            steps {
                container('build') {
                    echo "Running C unit tests..."
                    sh '''
                        make test

                        # Verify test results file exists
                        if [ -f test-results/c-test-results.xml ]; then
                            echo "C test results generated"
                            cat test-results/c-test-results.xml
                        else
                            echo "Warning: C test results file not found"
                        fi
                    '''
                }
            }
            // TODO: Uncomment the post block below to publish test results to CloudBees Unify
            // post {
            //     always {
            //         junit testResults: 'test-results/c-test-results.xml', allowEmptyResults: true
            //     }
            // }
        }

        stage('Run Python Unit Tests') {
            steps {
                container('build') {
                    echo "Running Python unit tests..."
                    sh '''
                        cd tests/python
                        pytest test_app.py \
                            --junitxml=../../test-results/pytest-results.xml \
                            --verbose
                    '''
                }
            }
            // TODO: Uncomment the post block below to publish test results to CloudBees Unify
            // post {
            //     always {
            //         junit testResults: 'test-results/pytest-results.xml', allowEmptyResults: true
            //     }
            // }
        }

        stage('Code Quality Analysis') {
            steps {
                container('build') {
                    echo "Running code quality checks..."
                    sh '''
                        # Python code quality
                        echo "=== Python Linting ==="
                        flake8 src/python/ --max-line-length=120 --statistics || true
                    '''
                }
            }
        }

        // TODO: Uncomment the stage below to add SAST Security Scan (SonarQube)
        // stage('SAST Security Scan - SonarQube') {
        //     when {
        //         expression { return fileExists('sonar-project.properties') }
        //     }
        //     steps {
        //         container('sonarqube') {
        //             echo "Running SonarQube SAST scan..."
        //             script {
        //                 sh '''
        //                     sonar-scanner \
        //                         -Dsonar.projectKey=${APP_NAME} \
        //                         -Dsonar.sources=src \
        //                         -Dsonar.host.url=${SONAR_HOST} \
        //                         -Dsonar.login=${SONAR_TOKEN}
        //                 '''
        //             }
        //         }
        //     }
        //     post {
        //         always {
        //             script {
        //                 exportSonarQubeScan(
        //                     component: "",
        //                     project: "${APP_NAME}",
        //                     host: "${SONAR_HOST}",
        //                     credentialId: "sonarqube-token"
        //                 )
        //             }
        //         }
        //     }
        // }

        // TODO: Uncomment the stage below to add SCA Security Scan (Trivy filesystem)
        // stage('SCA Security Scan - Trivy') {
        //     steps {
        //         container('trivy') {
        //             echo "Running Trivy security scan..."
        //             sh '''
        //                 # Run filesystem scan
        //                 echo "=== Trivy Filesystem Scan ==="
        //                 trivy fs --format sarif --output trivy-fs-report.sarif . --exit-code 0
        //
        //                 # Display scan summary
        //                 trivy fs --severity HIGH,CRITICAL . --exit-code 0
        //             '''
        //         }
        //     }
        //     post {
        //         always {
        //             script {
        //                 if (fileExists("trivy-fs-report.sarif")) {
        //                     registerSecurityScan(
        //                         artifacts: "trivy-fs-report.sarif",
        //                         format: "sarif",
        //                         scanner: "Trivy",
        //                         archive: true
        //                     )
        //                     echo "SCA scan results registered with CloudBees Unify"
        //                 }
        //             }
        //         }
        //     }
        // }

        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                container('build') {
                    echo "Simulating deployment to production..."
                    script {
                        sh """
                            echo "=== Deployment Configuration ==="
                            echo "Application: ${APP_NAME}"
                            echo "Version: ${APP_VERSION}"
                            echo "Artifact: ${APP_NAME}-${APP_VERSION}.tar.gz"

                            # Create deployment directory
                            mkdir -p deployment

                            # Extract application package
                            echo "Extracting application package..."
                            tar -xzf ${APP_NAME}-${APP_VERSION}.tar.gz -C deployment/

                            echo "Deployment directory prepared at: \${WORKSPACE}/deployment"
                            echo "Contents:"
                            ls -lh deployment/

                            # In a real deployment, you would:
                            # - Copy files to target server (scp, rsync)
                            # - Restart application services
                            # - Run health checks
                            # - Update load balancer configuration

                            echo "Deployment simulation complete!"
                        """
                    }
                }
            }
            // TODO: Uncomment the post block below to register deployment for DORA metrics
             post {
                 success {
                     script {
                         echo "Deployment to Production completed successfully"
            
                         // Register deployed artifact with CloudBees Unify
                         // This uses the artifact ID captured from registerBuildArtifactMetadata in Chapter 5
                         registerDeployedArtifactMetadata(
                             artifactId: buildArtifactId,
                             targetEnvironment: "Production",
                             labels: "deployed,deployment-${BUILD_NUMBER}"
                         )
            
                         echo "Deployment registered with CloudBees Unify for DORA metrics tracking"
                         echo "Environment: Production"
                         echo "Artifact ID: ${env.ARTIFACT_ID}"
                     }
                 }
                 failure {
                     echo "Deployment to Production failed"
                 }
             }
        }
    }

    post {
        always {
            echo "=== Pipeline Execution Complete ==="
            echo "Build: ${env.BUILD_NUMBER}"
            echo "Status: ${currentBuild.result}"
            echo "Duration: ${currentBuild.durationString}"

            // Archive test results (not published to CloudBees yet)
            archiveArtifacts artifacts: 'test-results/**/*.xml', allowEmptyArchive: true
            archiveArtifacts artifacts: 'build/**', allowEmptyArchive: true
        }

        success {
            echo "[SUCCESS] Pipeline completed successfully!"
        }

        failure {
            echo "[FAILURE] Pipeline failed"
            echo "Check the logs above for error details"
        }

        unstable {
            echo "[UNSTABLE] Pipeline completed with warnings"
        }
    }
}
