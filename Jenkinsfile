stage 'CI'
node {

   checkout scm

   //git branch: 'jenkins2-course', 
        //url: 'https://github.com/g0t4/solitaire-systemjs-course'

    // pull dependencies from npm
    // on windows use: bat 'npm install'
    sh 'npm install'

    // stash code & dependencies to expedite subsequent testing
    // and ensure same code & dependencies are used throughout the pipeline
    // stash is a temporary archive
    stash name: 'everything', 
          excludes: 'test-results/**', 
          includes: '**'
    
    // test with PhantomJS for "fast" "generic" results
    // on windows use: bat 'npm run test-single-run -- --browsers PhantomJS'
    sh 'npm run test-single-run -- --browsers PhantomJS'
    
    // archive karma test results (karma is configured to export junit xml files)
    step([$class: 'JUnitResultArchiver', 
          testResults: 'test-results/**/test-results.xml'])
          
}

//specify agent with label mac (see jenkins manage nodes)
node('mac') {
    
    sh 'ls' //list of all whats in the workspace
    sh 'rm -rf' //remove everything from workspace, so we can start from a emtpy workspace
    unstash 'everything'  //unstash the files we stahed before, see above on the other node
    sh 'ls'
    
}

//parallel test integration
stage 'Browser Testing'
//specifiy three "branches" (chrome, firefox, safari)
parallel chrome: {
    runTests("Chrome")
//}, firefox: {
    //runTests("Firefox")
}, safari: {
    runTests("Safari")
}

//function für runTests
def runTests(browser) {
    node {
        sh 'rm -rf *'
        unstash 'everything'
        sh "npm run test-single-run -- --browsers ${browser}"
        step([$class: 'JUnitResultArchiver',
                testResults: 'test-results/**/test-results.xml'])
    }
}

node {
    notify ("Deploy to staging?")
}

input 'Deploy to staging?' //outside of note, because otherwise a agent will be blocked until someboday will give the input --> flyweight executor on the master node is used, fw is always used if something is outside a node allocation

//limit concurrency so we dont perform simultaneous deploys
//and if multiple pipelines are executing,
// newest is only that will be allowed through, rest will be canceled
stage name: 'Deploy to staging', concurrency: 1
node {
    //hwo to deploy, nice to know
    //write build number to index page so we can see this update
    sh "echo '<h1>${env.BUILD_DISPLAY_NAME}<h1>' >> app/index.hmtl"
    
    // deploy to a docker container mapped to port 3000
    sh 'docker-compose up -d --build'
    
    notify 'Solitaire Deployed!'
}


def notify(status){
    emailext (
      to: "wesmdemos@gmail.com",
      subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
    )
}
