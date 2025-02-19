```
pipeline {
    agent any

    parameters {
        string(name: 'REMOTE_HOST', defaultValue: '', description: 'Enter the remote host IP or hostname')
        string(name: 'REMOTE_USER', defaultValue: '', description: 'Enter the SSH username')
        password(name: 'REMOTE_PASSWORD', defaultValue: '', description: 'Enter the SSH password')
    }

    environment {
        REMOTE_HOST = "${params.REMOTE_HOST}"
        REMOTE_USER = "${params.REMOTE_USER}"
        REMOTE_PASSWORD = "${params.REMOTE_PASSWORD}"
    }

    stages {
        stage('Validate Parameters') {
            steps {
                script {
                    if (!REMOTE_HOST || !REMOTE_USER || !REMOTE_PASSWORD) {
                        error("REMOTE_HOST, REMOTE_USER, and REMOTE_PASSWORD parameters are required.")
                    }
                }
            }
        }

        stage('SSH and Collect Metrics') {
            steps {
                script {
                    echo "Attempting SSH connection as user: ${REMOTE_USER} to host: ${REMOTE_HOST}"

                    // Capture the performance metrics securely
                    def metrics = ''
                    def timestamp = sh(script: "date +'%Y-%m-%d %H:%M:%S'", returnStdout: true).trim()

                    try {
                        // SSH into the remote machine using sshpass
                        metrics = sh(script: """
                            sshpass -p '${REMOTE_PASSWORD}' ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} << EOF
                            echo "CPU Usage (%)" && top -bn1 | grep "Cpu(s)" | awk '{print \$2 + \$4}'
                            echo "Memory Usage (MB)" && free -m | awk 'NR==2{printf "%.2f", \$3*100/\$2 }'
                            echo "Disk Usage" && df -h | awk '\$NF=="/"{print \$5}'
                            echo "Load Average" && uptime | awk -F 'load average:' '{ print \$2 }'
                            EOF
                        """, returnStdout: true).trim()
                    } catch (Exception e) {
                        error("Failed to fetch performance metrics: ${e}")
                    }

                    // Format metrics into CSV content
                    def csvContent = "Timestamp,Metric,Value\n"
                    metrics.split("\n").each { line ->
                        def parts = line.split(":")
                        if (parts.length == 2) {
                            def metric = parts[0].trim()
                            def value = parts[1].trim()
                            csvContent += "${timestamp},${metric},${value}\n"
                        }
                    }

                    // Write to CSV
                    writeFile file: 'performance_metrics.csv', text: csvContent
                }
            }
        }

        stage('Archive Results') {
            steps {
                archiveArtifacts artifacts: 'performance_metrics.csv', fingerprint: true
            }
        }
    }

    post {
        always {
            echo "Pipeline execution completed."
        }
        failure {
            echo "Pipeline failed. Check logs for details."
        }
    }
}
```
