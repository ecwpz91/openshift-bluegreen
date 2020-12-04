import java.text.SimpleDateFormat;

// This pipeline expects a paramater newcolor which will set the newcolor of the newly deployed application.

node {
  // Blue/Green Deployment into Production
  // -------------------------------------
  def project  = ""
  def dest     = "example-green"
  def active   = ""
  def newcolor = ""

  switch (BUILD_NUMBER.toInteger() % 10) {
    case 1:
        newcolor="red"
        break
    case 2:
        newcolor="maroon"
        break
    case 3:
        newcolor="yellow"
        break
    case 4:
        newcolor="magenta"
        break
    case 5:
        newcolor="purple"
        break
    case 6:
        newcolor="orange"
        break
    case 7:
        newcolor="cyan"
        break
    case 8:
        newcolor="navy"
        break
    case 9:
        newcolor="green"
        break
    case 0:
        newcolor="blue"
        break
  }

  stage('Determine Deployment color') {
    // Determine current project
    sh "oc get project|grep -v NAME|awk '{print \$1}' >project.txt"
    project = readFile('project.txt').trim()
    sh "oc get route example -n ${project} -o jsonpath='{ .spec.to.name }' > activesvc.txt"

    // Determine currently active Service
    active = readFile('activesvc.txt').trim()
    if (active == "example-green") {
      dest = "example-blue"
    }
    echo "Active svc: " + active
    echo "Dest svc:   " + dest
    echo "New color:  " + newcolor
  }

  stage('Build new version') {
    // Building in this case means simply changing the environment
    // variable newcolor in the deployment configuration.
    // There is not Build Configuration since this is a straight
    // up Docker Image deployment.
    echo "Building ${dest}"
    sh "oc set env dc ${dest} COLOR=${newcolor}"
  }

  stage('Deploy new Version') {
    echo "Deploying to ${dest}"
    
    openshift.withCluster() {
        openshift.withProject() {
          def rm = openshift.selector("dc", dest).rollout()
          timeout(5) { 
            openshift.selector("dc", dest).related('pods').untilEach(1) {
              return (it.object().status.phase == "Running")
            }
          }
        }
    }
  }
  
  stage('Switch over to new Version') {
    input "Switch Production?"
    sh 'oc patch route example -p \'{"spec":{"to":{"name":"' + dest + '"}}}\''
    sh 'oc get route example > oc_out.txt'
    oc_out = readFile('oc_out.txt')
    echo "Current route configuration: " + oc_out
  }
}
