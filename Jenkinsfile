pipeline {
    agent any
    
    tools {
        go 'Go 1.16'
    }
    
    stages {
        stage ("Test compile") {
            steps ("Download and test") {
                sh '''
                    make lint && make test
                    if [ $? -ne 0 ];
                        then
                            echo "Test error"
                            exit 1
                    fi
                '''
            }
        }
         stage ("Build"){
            steps{
                sh '''export GOPATH=$WORKSPACE/go
                export PATH="$PATH:$(go env GOPATH)/bin"               
                go get github.com/tools/godep
                go get github.com/smartystreets/goconvey
                go get github.com/GeertJohan/go.rice/rice  
                go get github.com/wickett/word-cloud-generator/wordyapi
                go get github.com/gorilla/mux
                sed -i "s/1.DEVELOPMENT/1.$BUILD_NUMBER/g" static/version
                GOOS=linux GOARCH=amd64 go build -o ./artifacts/word-cloud-generator -v 
                md5sum artifacts/word-cloud-generator
                ls -l artifacts/
                gzip -f ./artifacts/word-cloud-generator'''
            }
        }
        stage ("Upload to nexus") {
            steps {
                nexusArtifactUploader (artifacts: [[artifactId: 'world-cloud-generator', 
                classifier: '', file: 'artifacts/word-cloud-generator.gz', 
                type: 'gz']], credentialsId: 'uploader', groupId: "$git_branch", 
                nexusUrl: '192.168.56.20:8081', nexusVersion: 'nexus3', protocol: 'http', 
                repository: 'world-cloud-build', version: '1.$BUILD_NUMBER')
            }
        }   
        stage ("Paralleled test") {
            parallel {
                stage ("Test and install on stading") {
                    steps {
                        sh '''
                              sshpass -p 'vagrant' ssh vagrant@192.168.56.30 -o StrictHostKeyChecking=no "cd /opt/wordcloud/
                              sudo curl -u downloader:123 -X GET "http://192.168.56.20:8081/repository/world-cloud-build/$git_branch/world-cloud-generator/1.$BUILD_NUMBER/world-cloud-generator-1.$BUILD_NUMBER.gz" -o /opt/wordcloud/word-cloud-generator.gz
                                                                 
                              if [[ $? -ne 0 ]];
                                then
                                    echo "File not found"
                                    exit 1
                                else
                                  sudo service wordcloud stop
                                  sudo gunzip -f /opt/wordcloud/word-cloud-generator.gz
                                  sudo chmod +x /opt/wordcloud/word-cloud-generator
                                  sudo service wordcloud start
                                fi"
                           '''
                    }
                }
                stage ("Test and install on prodaction") {
                    steps {
                        sh '''
                              sshpass -p 'vagrant' ssh vagrant@192.168.56.40 -o StrictHostKeyChecking=no "cd /opt/wordcloud/
                              sudo curl -u downloader:123 -X GET "http://192.168.56.20:8081/repository/world-cloud-build/$git_branch/world-cloud-generator/1.$BUILD_NUMBER/world-cloud-generator-1.$BUILD_NUMBER.gz" -o /opt/wordcloud/word-cloud-generator.gz
                              if [[ $? -ne 0 ]];
                                then
                                    echo "File not found"
                                    exit 1
                                else
                                  sudo service wordcloud stop
                                  sudo gunzip -f /opt/wordcloud/word-cloud-generator.gz
                                  sudo chmod +x /opt/wordcloud/word-cloud-generator
                                  sudo service wordcloud start
                                fi"
                           '''
                    }
                }
            }
        }
        
    }
}
