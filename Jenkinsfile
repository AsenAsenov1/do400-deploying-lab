pipeline {
    agent {
        node { label "maven" }
    }
    environment { QUAY_USR = credentials('QUAY_USER') }

    stages {
        stage("Test") {
            steps {
                sh "./mvnw verify"
            }
        }
        stage("Build & Push Image") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'QUAY_USER', usernameVariable: 'QUAY_USERNAME', passwordVariable: 'QUAY_PASSWORD')]) {
                    sh '''
                    ./mvnw quarkus:add-extension \
                    -Dextensions="container-image-jib"
                    '''
                    sh '''
                    ./mvnw package -DskipTests \
                    -Dquarkus.jib.base-jvm-image=quay.io/redhattraining/do400-java-alpine-openjdk11-jre:latest \
                    -Dquarkus.container-image.build=true \
                    -Dquarkus.container-image.registry=quay.io \
                    -Dquarkus.container-image.group=${QUAY_USERNAME} \
                    -Dquarkus.container-image.name=do400-deploying-lab \
                    -Dquarkus.container-image.username=${QUAY_USERNAME} \
                    -Dquarkus.container-image.password=${QUAY_PASSWORD} \
                    -Dquarkus.container-image.tag=build-${BUILD_NUMBER} \
                    -Dquarkus.container-image.additional-tags=latest \
                    -Dquarkus.container-image.push=true
                    '''
                }
            }
        }
        stage("Deploy to Test") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'QUAY_USER', usernameVariable: 'QUAY_USERNAME', passwordVariable: 'QUAY_PASSWORD')]) {
                    sh """
                    oc set image deployment home-automation \
                    home-automation=quay.io/${QUAY_USERNAME}/do400-deploying-lab:build-${BUILD_NUMBER} \
                    -n RHT_OCP4_DEV_USER-deploying-lab-test --record
                    """
                }
            }
        }
    }
}
