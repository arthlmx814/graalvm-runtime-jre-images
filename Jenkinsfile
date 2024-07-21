/*
 * Jenkinsfile for building and publishing GraalVM JRE images.
 * URL: https://jenkins.arthlmx.fr/job/arthlmx814/job/graalvm-runtime-jre-images/
 */

/* Functions definition */

/**
 * Builds a GraalVM JRE image
 *
 * @param javaVersion Java version (17, 21, 22, etc.)
 */
def buildGraalVMJreImage(String javaVersion) {
    // Fetch the GraalVM JDK image
    def graalVMJDKImage = docker.image("container-registry.oracle.com/graalvm/jdk:${javaVersion}")

    // Pull the GraalVM JDK image
    graalVMJDKImage.pull()
    // Inside the GraalVM JDK image, get the Java full version and save it to a file (e.g., 17.0.10)
    graalVMJDKImage.inside {
        sh "/bin/bash -c 'cat \$JAVA_HOME/release' | grep 'JAVA_VERSION=' | cut -d '=' -f 2 | tr -d '[:space:]' | tr -d '\"' > .java_version"
    }

    try {
        // Read the Java version from the file
        def javaFullVersion = readFile('.java_version').trim()

        // Connect to the Docker registry and build the GraalVM JRE image
        docker.withRegistry(env.DOCKER_REGISTRY, env.DOCKER_CREDENTIAL_ID) {
            // Create the docker image based on the docker registry
            def image = docker.image(env.IMAGE_NAME)
            // The tags to add to the final image
            def tags = [
                javaFullVersion,
                javaVersion
            ]
            // The labels to add to the final image
            def labels = [
                "org.opencontainers.image.title='graalvm-jre-${javaVersion}'",
                "org.opencontainers.image.description='GraalVM JRE ${javaVersion} image based on Ubuntu'",
                "org.opencontainers.image.version='${javaFullVersion}'",
                "org.opencontainers.image.url='https://github.com/arthlmx814/graalvm-runtime-jre-images'",
                "org.opencontainers.image.documentation='https://github.com/arthlmx814/graalvm-runtime-jre-images/blob/main/README.md'",
                "org.opencontainers.image.licenses='MIT'"
            ]

            // Build the GraalVM JRE image
            sh """
                docker build \
                    ${tags.collect { "--tag=${image.imageName()}:${it}" }.join(' ')} \
                    --pull \
                    --progress=plain \
                    --platform=linux/arm64,linux/amd64 \
                    --build-arg JAVA_VERSION=${javaFullVersion} \
                    --output=type=image,push=true,registry.insecure=true \
                    ${labels.collect { "--label=${it}" }.join(' ')} \
                    ./${javaVersion}
            """
        }
    } finally {
        // Clean up the generated file
        sh 'rm -f .java_version'
    }
}

/* Pipeline definition */
pipeline {
    agent {
        // Run the pipeline on a unix (linux or mac os) agent running with Docker installed
        label 'linux && docker'
    }

    options {
        // Enable timestamps for the console output
        timestamps()
        // Add a timeout for the whole pipeline (1 hour)
        timeout(time: 1, unit: 'HOURS')
        // Do not allow concurrent builds
        disableConcurrentBuilds(abortPrevious: true)
        // Rotate the build logs to keep the last 20 builds
        buildDiscarder(logRotator(numToKeepStr: '20'))
        // Do not automatically checkout the code
        skipDefaultCheckout()
    }

    environment {
        // Set the Git credentials ID
        GIT_CREDENTIAL_ID = 'a8f1cbad-2d3c-4190-aa0f-2a3f7a9a464f'
        // Set the Docker credentials ID
        DOCKER_CREDENTIAL_ID = '07856d6d-feed-4e1f-812e-64e0d4d7ce40'
        // Set the Docker registry URL
        DOCKER_REGISTRY = 'https://registry.hub.docker.com'
        // Set the Docker image name
        IMAGE_NAME = 'arthlmx814/graalvm-jre'
        // Set the senders email
        MAIL_TO = 'notify-arthlmx814@jenkins.arthlmx.fr'
    }

    parameters {
        string(
            name: 'GIT_BRANCH',
            defaultValue: 'main',
            description: 'Git branch to checkout',
            trim: true
        )
    }

    stages {
        stage('Checkout') {
            steps {
                git(
                    branch: params.GIT_BRANCH,
                    credentialsId: env.GIT_CREDENTIAL_ID,
                    url: 'git@github.com:arthlmx814/graalvm-runtime-jre-images.git'
                )
            }
        }
        stage('Build and Publish GraalVM JRE Images') {
            when {
                anyOf {
                    expression {
                        env.BUILD_NUMBER == '1' // First build
                    }
                    changeset 'Jenkinsfile' // Jenkinsfile changes
                    changeset '**/Dockerfile' // Dockerfile changes
                    changeset '**/jmods.list' // jmods.list changes
                    triggeredBy 'TimerTrigger' // Triggered by a scheduled build
                    triggeredBy 'UserIdCause' // Triggered by a manual build
                }
            }
            stages {
                stage('GraalVM 17') {
                    steps {
                        buildGraalVMJreImage('17')
                    }
                }
                stage('GraalVM 21') {
                    steps {
                        buildGraalVMJreImage('21')
                    }
                }
                stage('GraalVM 22') {
                    steps {
                        buildGraalVMJreImage('22')
                    }
                }
            }
        }
    }

    post {
        fixed {
            // Send an email notification on fixed builds
            mail(
                to: env.MAIL_TO,
                subject: "<${env.JOB_NAME}> - Build #${env.BUILD_NUMBER} - FIXED",
                body: """
                    <p>Build #${env.BUILD_NUMBER} of ${env.JOB_NAME} has been fixed.</p>
                    <br>
                    <p>Check the build details at <a href="${env.BUILD_URL}">${env.BUILD_URL}</a>.</p>
                """,
                mimeType: 'text/html'
            )
        }

        failure {
            // Send an email notification on failed builds
            mail(
                to: env.MAIL_TO,
                subject: "<${env.JOB_NAME}> - Build #${env.BUILD_NUMBER} - FAILURE",
                body: """
                    <p>Build #${env.BUILD_NUMBER} of ${env.JOB_NAME} has failed.</p>
                    <br>
                    <p>Check the build details at <a href="${env.BUILD_URL}">${env.BUILD_URL}</a>.</p>
                """,
                mimeType: 'text/html'
            )
        }
    }
}
