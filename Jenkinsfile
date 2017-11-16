@Library("PTCSLibrary@savpek") _

// Podtemplate and node must match, dont use generic names like 'node', use more specific like projectname or node + excact version number.
// This is because CI environment reuses templates based on naming, if you create node 7 environment with name 'node', following node 8 environment
// builds may fail because they reuse same environment if label matches existing.
podTemplate(label: 'beertapped', idleMinutes:30,
  containers: [
    containerTemplate(name: 'node', image: 'node:7.10.0', ttyEnabled: true, command: '/bin/sh -c', args: 'cat'),
    containerTemplate(name: 'docker', image: 'ptcos/docker-client:latest', alwaysPullImage: true, ttyEnabled: true, command: '/bin/sh -c', args: 'cat')
  ]
) {
    def project = 'beertapped'
    def branch = (env.BRANCH_NAME)
    def namespace = "beertapped"

    node('beertapped') {
        stage('Checkout') {
            checkout_with_tags()
        }
        stage('Build') {
            container('node') {
                // sh """
                //     npm install -g @angular/cli
                //     npm install
                //     ng build --prod
                // """
            }
        }
        stage('Test') {
            container('node') {
                // sh """
                //     npm run test
                // """
            }
        }
        stage('Package') {
            container('docker') {
                def published = publishContainerToGcr(project, branch);

                toK8sTestEnv() {
                    sh """
                        kubectl apply -f ./k8s/${branch}.yaml
                        kubectl set image deployment/$project-$branch $project-$branch=$published.image:$published.tag --namespace=$namespace
                    """
                }
            }
        }
    }
  }