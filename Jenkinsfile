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
					def scanFile = "nmap_scan_${env.TIMESTAMP}.txt"
					sh """
						ls ${env.WORKSPACE} > test.txt
						apt-get update && apt-get install -y nmap > /dev/null
						ip=\$(hostname -I | awk '{print \$1}')
						echo "Escaneando con Nmap la IP local: \$ip" > ${env.OUTPUT_DIR}/${scanFile}
						nmap -sS -T4 \$ip >> ${env.OUTPUT_DIR}/${scanFile}
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
