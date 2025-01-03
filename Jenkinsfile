pipeline {
    agent {
        docker {
            image 'ubuntu:20.04'  
            args '-v /var/run/docker.sock:/var/run/docker.sock' 
        }
    }

    environment {
        YOCTO_DIR = '/poky'  
        REPO_URL = 'https://github.com/allanmpb/build_opi_jenkins.git' 
    }

    stages {
        stage('Clonando Repositório Git') {
            steps {
                script {
                    // Clona o repositório do Git
                    git url: "$REPO_URL", branch: 'master', credentialsId: 'git-credentials-id' 
                }
            }
        }

        stage('Preparação do Ambiente') {
            steps {
                script {
                    // Instala o pacote tzdata antes de configurar o fuso horário
                    sh 'apt-get update'
                    sh 'apt-get install -y tzdata'  // Instala o tzdata para configurar o fuso horário
                    sh 'echo "Etc/UTC" > /etc/timezone'
                    sh 'dpkg-reconfigure --frontend noninteractive tzdata'  // Configura o fuso horário sem interação

                    sh '''
                    apt-get install -y \
                        gawk wget git-core diffstat unzip texinfo gcc-multilib build-essential \
                        chrpath socat libsdl1.2-dev xterm lz4 python3 python3-pip sudo \
                        ca-certificates curl openjdk-11-jdk docker.io
                    '''
                }
            }
        }

        stage('Clonando Repositórios Yocto') {
            steps {
                script {                    
                    sh 'git clone -b kirkstone git://git.yoctoproject.org/poky.git /poky'
                    sh 'git clone -b kirkstone git://git.openembedded.org/meta-openembedded.git /poky/meta-openembedded'
                    sh 'git clone -b kirkstone https://github.com/linux-sunxi/meta-sunxi.git /poky/meta-sunxi'
                    sh 'git clone -b 6.4.2 git://code.qt.io/yocto/meta-qt6.git /poky/meta-qt6'
                }
            }
        }

        stage('Configuração do Yocto') {
            steps {
                script {
                    sh 'source /poky/oe-init-build-env'

                    sh '''
                    echo 'MACHINE = "orange-pi-pc-plus"' >> /poky/build/conf/local.conf
                    echo 'BBLAYERS += "/poky/meta /poky/meta-poky /poky/meta-yocto-bsp /poky/meta-openembedded/meta-oe /poky/meta-openembedded/meta-networking /poky/meta-openembedded/meta-python /poky/meta-sunxi /poky/meta-qt6"' >> /poky/build/conf/bblayers.conf
                    '''

                    sh '''
                    echo 'IMAGE_INSTALL:append = " qtbase qtdeclarative qtdeclarative-tools qttools qttranslations-qtbase qttranslations-qtdeclarative"' >> /poky/build/conf/local.conf
                    '''
                }
            }
        }

        stage('Construção da Imagem') {
            steps {
                script {
                    sh 'bitbake core-image-minimal'
                }
            }
        }

        stage('Finalizando Build') {
            steps {
                script {   
                    sh 'cp /poky/tmp/deploy/images/orange-pi-pc-plus/core-image-minimal-orangepi-pc-plus.wic /poky/output.wic'
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
