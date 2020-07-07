node {
    def app

    stage('Clone repository') {
        /* Let's make sure we have the repository cloned to our workspace */

        checkout scm
    }

    stage('Build image') {
        /* This builds the actual image; synonymous to
         * docker build on the command line */

        app = docker.build("doliva/vote")
    }


    stage('Initialize') {
        def dockerHome = tool 'myDocker'
        env.PATH = "${dockerHome}/bin:${env.PATH}"
    }

    stage('Test image') {
        app.inside {
	    # sh 'echo "Run container and test access "'
                sh '''
		    docker run -d \
		    --rm \
		    -u root \
	     	    -p 8060:80 \
		    -v "$PWD":/opt/bitnami/jenkins/jenkins_home \
		    -v /var/run/docker.sock:/var/run/docker.sock \
		    doliva/vote:latest

		    /bin/sleep 0.2

		    /usr/bin/curl -I localhost:8060 -m 2

		    /bin/sleep 0.2
		    /usr/bin/docker stop $(/usr/bin/docker ps -a -q)
		    /usr/bin/docker rm $(/usr/bin/docker ps -a -q)

                '''
	    # sh 'echo "Test validated. Delete environment "'
        }
    }

    stage('Push image') {
        /* Finally, we'll push the image with two tags:
         * First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */
        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
            app.push("${env.BUILD_NUMBER}")
            app.push("latest")
        }
    }
}
