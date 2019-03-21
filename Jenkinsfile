jettyUrl = 'http://localhost:8081/'
servers = library('tools').demo.Servers.new(this)

stage('Dev') {
    node {
        checkout scm
        mvn '-o clean package'
        dir('target') {stash name: 'war', includes: 'x.war'}
        node {
            servers.deploy env.BRANCH_NAME
        }
        echo "Deployed to ${jettyUrl}$env.BRANCH_NAME/"
    }
}

stage('QA') {
    echo "Skip doing tests"
}

stage('Staging') {
    if(env.BRANCH_NAME == "master"){
        lock(resource: 'staging-server', inversePrecedence: true) {
            input message: "Do you want to deploy to staging? ${jettyUrl}staging/"
            node {
                servers.deploy 'staging'
            }
        }
        try {
            checkpoint('Before production')
        } catch (NoSuchMethodError _) {
            echo 'Checkpoint feature available in CloudBees Jenkins Enterprise.'
        }
    }

}

stage ('Production') {
    if(env.BRANCH_NAME == "master"){
        lock(resource: 'production-server', inversePrecedence: true) {
            input message: "Do you want to deploy to production? ${jettyUrl}production/"
            node {
                sh "wget -O - -S ${jettyUrl}staging/"
                echo 'Production server looks to be alive'
                servers.deploy 'production'
                echo "Deployed to ${jettyUrl}production/"
            }
        }
    }
}
