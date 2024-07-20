/*
 * Jenkinsfile for building and publishing GraalVM JRE images
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
                    --output=type=image,push=true \
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
        always {
            // Clean up the workspace
            cleanWs()
        }
    }
}
