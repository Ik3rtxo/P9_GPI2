pipeline {
    agent any

    environment {
        TIMESTAMP = "${new Date().format('yyyyMMdd_HHmmss')}"
        OUTPUT_DIR = "security_audit_${env.TIMESTAMP}"
        TARGET_PATH = "${env.WORKSPACE}/src/index.js"
    }

    stages {

        stage('Preparar Directorio') {
            steps {
                sh 'mkdir -p $OUTPUT_DIR'
            }
        }

        stage('Versiones del Sistema') {
            steps {
                script {
                    def filePath = "${env.OUTPUT_DIR}/system_info_${env.TIMESTAMP}.txt"
                    sh """
                        echo 'Java Version:' > "${filePath}"
                        java -version 2>&1 | tee -a "${filePath}"

                        echo '\\nJenkins Version:' >> "${filePath}"
                        curl -sI http://localhost:8080 | grep -i 'X-Jenkins' | grep -v 'Session' | awk '{print \$2}' >> "${filePath}" || echo 'No se pudo obtener versión Jenkins'
                    """
                }
            }
        }

        stage('Escaneo de Puertos con Nmap') {
            steps {
                script {
                    def ip = sh(script: "hostname -I | awk '{print \$1}'", returnStdout: true).trim()
                    def scanFile = "port_scan_${env.TIMESTAMP}.txt"
                    env.SCAN_FILE = scanFile  // Si quieres usarlo globalmente más tarde
                    sh """
                        echo "Escaneando con Nmap (modo sin root) la IP local: ${ip}" > ${env.OUTPUT_DIR}/${scanFile}
                        nmap -sT -T4 ${ip} >> ${env.OUTPUT_DIR}/${scanFile} || echo 'Nmap no está instalado o falló.'
                    """
                }
            }
        }

        stage('SHA-256 de Archivos') {
            steps {
                script {
                    def shaFile = "hashes_${env.TIMESTAMP}.txt"
                    sh """
                        echo "Generando hashes SHA-256 de los archivos..." > ${env.OUTPUT_DIR}/${shaFile}
                        find ${env.OUTPUT_DIR} -type f -exec sha256sum {} \\; >> ${env.OUTPUT_DIR}/${shaFile}
                    """
                }
            }
        }
    }

    post {
        always {
            echo "Auditoría de seguridad guardada en: ${env.OUTPUT_DIR}"
        }
    }
}
