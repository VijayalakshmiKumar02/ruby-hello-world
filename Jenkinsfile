pipeline
{
    agent{
      label 'nodejs'
          }
            stages{
            stage('preamble'){
                steps{
                    script{
                openshift.withCluster() {
                    openshift.withProject() {
                        echo "Using project: ${openshift.project()}"
                    }
                }
            }
                }
            }
            stage('cleanup'){
                steps{
                    script{
                 openshift.withCluster() {
                    openshift.withProject() {
                        
                        def templateName = 'ruby-hello-world'
                       openshift.selector("all", [ app : templateName ]).delete() 
                  if (openshift.selector("secrets", templateName).exists()) { 
                    openshift.selector("secrets", templateName).delete()
                  
                  }
                    }
                }
            }
                }
            }
            stage('Create Nodejs App'){
                steps{
                    script{
                openshift.withCluster() {
                    openshift.withProject() {
                    def bc = openshift.newApp( 'https://github.com/VijayalakshmiKumar02/ruby-hello-world' ).narrow('bc')

    // The build config will create a new build object automatically, but how do
    // we find it? The 'related(kind)' operation can create an appropriate Selector for us.
    def builds = bc.related('builds')

    // There are no guarantees in life, so let's interrupt these operations if they
    // take more than 10 minutes and fail this script.
    timeout(10) {

        // We can use watch to execute a closure body each objects selected by a Selector
        // change. The watch will only terminate when the body returns true.
        builds.watch {
            // Within the body, the variable 'it' is bound to the watched Selector (i.e. builds)
            echo "So far, ${bc.name()} has created builds: ${it.names()}"

            // End the watch only once a build object has been created.
            return it.count() > 0
        }

        // But we can actually want to wait for the build to complete.
        builds.watch {
            if ( it.count() == 0 ) return false

            // A robust script should not assume that only one build has been created, so
            // we will need to iterate through all builds.
            def allDone = true
            it.withEach {
                // 'it' is now bound to a Selector selecting a single object for this iteration.
                // Let's model it in Groovy to check its status.
                def buildModel = it.object()
                if ( it.object().status.phase != "Complete" ) {
                    allDone = false
                }
            }

            return allDone;
        }


        // Uggh. That was actually a significant amount of code. Let's use untilEach(){}
        // instead. It acts like watch, but only executes the closure body once
        // a minimum number of objects meet the Selector's criteria only terminates
        // once the body returns true for all selected objects.
        builds.untilEach(1) { // We want a minimum of 1 build

            // Unlike watch(), untilEach binds 'it' to a Selector for a single object.
            // Thus, untilEach will only terminate when all selected objects satisfy this
            // the condition established in the closure body (or until the timeout(10)
            // interrupts the operation).

            return it.object().status.phase == "Complete"
        }
                    }
                }
            }
            }
                }
            }
    stage('deployment'){
        steps{
            script{
                 openshift.withCluster() {
                    openshift.withProject() {
                        def dc = openshift.selector("dc", "ruby-hello-world")
                            echo "Calling deployment rollout"
                            dc.rollout().status()
                          }
                    }
                }
        }
    }
        
                stage('expose service'){
                    steps{
                        script{
                        openshift.withCluster() {
                    openshift.withProject() {
                       echo "Calling expose service."
                         def svc = openshift.selector('svc', "ruby-hello-world")
                        svc.expose()
                        echo "Service exposed"
                    }
                        }
                        }
                    }
                }
}
}
