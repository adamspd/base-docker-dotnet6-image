pipeline {
    // Limit this to run on only some machines
    agent { label "MCSRND02" }
    
    
    stages {
        stage("Tag sources") {
            steps {
                script {
                    withEnv(["GITTAG=${ASSEMBLY_VERSION}.${BUILD_NUMBER}"]) {
                        withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'jenkins-bitbucket-common-creds', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD']]) {
                            String repo = env.GIT_URL.replace('https://','')
                            bat "git restore & git tag %GITTAG% "
                            bat "git push https://${env.GIT_USERNAME}:${env.GIT_PASSWORD}@${repo} %GITTAG%"
                        }
                    }
                }
            }
        }

        stage('Docker') {
			agent{ label "docker" }
            steps {
                echo 'Build for docker starts'

                script {
                    def app
                    stage ('Checkout') {
                        checkout scm
                    }

                    stage('Getting the timestamp') {
                        script {
                            DATE_TAG = java.time.LocalDate.now()
                            DATETIME_TAG = java.time.LocalDateTime.now()
                        }
                        sh "echo ${DATE_TAG}"
                    }

	
                    def buildargs = ". -f ./build/Dockerfile --build-arg BUILD_TIMESTAMP=${DATE_TAG} --build-arg BUILD_VERSION=${env.ASSEMBLY_VERSION}.${env.BUILD_NUMBER}"
                    def runtimeargs = ". -f ./runtime/Dockerfile --build-arg BUILD_TIMESTAMP=${DATE_TAG} --build-arg BUILD_VERSION=${env.ASSEMBLY_VERSION}.${env.BUILD_NUMBER}"
                    

                    stage('Build applications and images in dockerfile') {
                        docker.withRegistry('https://172.17.145.37', 'harbor-server-docker') {
                            withEnv(["RELEASE_BUILD=-p:Configuration=Release -p:VersionPrefix=%ASSEMBLY_VERSION% -P:InformationalVersion=%ASSEMBLY_VERSION%.%BUILD_NUMBER% -p:Company=\"%COMPANY%\" -p:CopyRight=\"%ASSEMBLY_COPYRIGHT%\""]) {
                                app1 = docker.build("build/modulus-dotnet6-build-image", buildargs)
                                app2 = docker.build("build/modulus-dotnet6-runtime-image", runtimeargs)
                            }
                        }
                    }

                    stage ('Push to registry') {
                        // first argument is the ip address of our harbor server (obvious), second, the credentials to connect to it stored
                        // in our jenkins server under: Manage Jenkins > Manage Credentials
                        docker.withRegistry('https://172.17.145.37', 'harbor-server-docker') {

                            // push version tags
                            app1.push("${env.ASSEMBLY_VERSION}.${env.BUILD_NUMBER}")
                            app2.push("${env.ASSEMBLY_VERSION}.${env.BUILD_NUMBER}")
                        }
                    }

                    // needed by BuildScript/docker/archive_images.sh
                    stage("Export tag to file") {
                        sh ("""mkdir -p /tmp/last_docker_imgs_tags/ ; echo ${env.ASSEMBLY_VERSION}.${env.BUILD_NUMBER} > /tmp/last_docker_imgs_tags/aml""")
                    }

                    stage("Cleanup") {
                        sh ("""docker image prune -a -f""")
                        sh ("""docker system prune -a -f""")
                    }
                }
            }
        }
    }
}