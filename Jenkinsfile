node() {
checkout scm
stage('build the image') {
withDockerServer([credentialsId: 'mymachine', uri:
"tcp://${DOCKERHOST}:2376"]) {
docker.build "${DOCKER_REGISTRY_USER}/rsvpapp:mooc"
}
}

stage('push the image to DockerHub') {
withDockerServer([credentialsId: 'mymachine', uri: "tcp://${DOCKERHOST}:2376"])
{
withDockerRegistry([credentialsId: 'dockerhub_auth']) {
docker.image("${DOCKER_REGISTRY_USER}/rsvpapp:mooc").push()
}
}
}

stage('deploy the image to staging server') {

withEnv(['COMPOSE_TLS_VERSION=TLSv1_2']) {
    withDockerServer([credentialsId: 'staging-server', uri:
"tcp://${STAGING}:2376"]){
sh '/usr/local/bin/docker-compose pull'
sh '/usr/local/bin/docker-compose -p rsvp_staging up -d'
}
input 'Check application running at http://130.193.44.27:5000 Looks good ?'
withDockerServer([credentialsId: 'staging-server', uri:
"tcp://${STAGING}:2376"]) {
sh '/usr/local/bin/docker-compose -p rsvp_staging down -v'
}
}
}

stage('deploy in production'){
withDockerServer([credentialsId: 'production', uri: "tcp://${PRODUCTION}:2376"]) {
sh 'docker stack deploy -c docker-stack.yaml myrsvpapp'
}
input 'Check application running at http://130.193.35.197:5000'
withDockerServer([credentialsId: 'production', uri: "tcp://${PRODUCTION}:2376"]) {
sh 'docker stack down myrsvpapp'
}
}

}
