node {
    def app

    stage('Clone repository') {
        /* Let's make sure we have the repository cloned to our workspace */

        checkout scm
    }

    stage('Build image') {
        /* This builds the actual image; synonymous to
         * docker build on the command line */

        app = docker.build("finalproj20/bwapp_docker")
    }

    stage('Push image') {
        /* Finally, we'll push the image with two tags:
         * First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */
        docker.withRegistry('https://registry.hub.docker.com', 'dockerHub') {
            app.push("${env.BUILD_NUMBER}")
            app.push("latest")
        }
    }
    
     /*stage('Aqua Scan') {
            aquaMicroscanner imageName: 'finalproj20/bwapp_docker', notCompliesCmd: '', onDisallowed: 'ignore', outputFormat: 'html'
    }*/
    
   stage('Anchore Scan') {
        def imageLine = 'finalproj20/bwapp_docker'
        writeFile file: 'anchore_images', text: imageLine
        anchore name: 'anchore_images'
    } 
    
    docker.image('finalproj20/bwapp_docker').withRun('-p 8000:80') {
            stage('Setting Up the App'){
                script {
                  env.TAG_PROGRESS = input message: 'User input required',
                      parameters: [choice(name: 'TAG_PROGRESS', choices: 'no\nyes', description: 'Choose "yes" if the pipeline can continue')]
                }
            }
                
            stage('Arachni') {
            sh '''
                mkdir -p $PWD/reports $PWD/artifacts;
                docker run \
                    -v $PWD/reports:/arachni/reports ahannigan/docker-arachni \
                    bin/arachni http://172.17.0.1:8000 --report-save-path=reports/example.io.afr;
                docker run --name=arachni_report  \
                    -v $PWD/reports:/arachni/reports ahannigan/docker-arachni \
                    bin/arachni_reporter reports/example.io.afr --reporter=html:outfile=reports/example-io-report.html.zip;
                docker cp arachni_report:/arachni/reports/example-io-report.html.zip $PWD/artifacts;
                docker rm arachni_report;
            '''
            archiveArtifacts artifacts: 'artifacts/**', fingerprint: true
            }
            
            /*stage('BurpSuite Scan') {
                        build job: 'Burp-BWAPP', parameters: [
                        string(name: 'true')
                        ]
                    
            }*/
        
        }
}
