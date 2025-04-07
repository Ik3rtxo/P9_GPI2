pipeline {
    agent any

    environment {
        TIMESTAMP = "${new Date().format('yyyyMMdd_HHmmss')}"
        OUTPUT_DIR = "security_audit_${env.TIMESTAMP}"
        TARGET_PATH = "${env.WORKSPACE}/src/actions/index.js"
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
                    def fileName = "system_info_${env.TIMESTAMP}.txt"
                    sh """
                        echo 'Java Version:' > ${env.OUTPUT_DIR}/${fileName}
                        java -version 2>&1 | tee -a ${env.OUTPUT_DIR}/${fileName}
                        
                        echo '\\nJenkins Version:' >> ${env.OUTPUT_DIR}/${fileName}
                        curl -sI http://localhost:8080 | grep -i 'X-Jenkins' | grep -v 'Session' | awk '{print $2}' >> ${env.OUTPUT_DIR}/${fileName} || echo 'No se pudo obtener versión Jenkins'
                    """
                }
            }
        }

        stage('Escaneo de Puertos Locales') {
            steps {
                script {
                    def scanFile = "port_scan_${env.TIMESTAMP}.txt"
                    sh """
                        ip=\$(hostname -I | awk '{print \$1}')
                        echo "Escaneando IP local: \$ip" > ${env.OUTPUT_DIR}/${scanFile}
                        nc -zv \$ip 1-1024 2>&1 | grep succeeded >> ${env.OUTPUT_DIR}/${scanFile} || echo 'No hay puertos abiertos'
                    """
                }
            }
        }

        stage('SHA-256 de Archivos') {
            steps {
                script {
                    def shaFile = "hashes_${env.TIMESTAMP}.txt"
                    sh """
                        if [ -d "${env.TARGET_PATH}" ]; then
                            find "${env.TARGET_PATH}" -type f -exec sha256sum {} \\; > ${env.OUTPUT_DIR}/${shaFile}
                        else
                            echo "Ruta no encontrada: ${env.TARGET_PATH}" > ${env.OUTPUT_DIR}/${shaFile}
                        fi
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
