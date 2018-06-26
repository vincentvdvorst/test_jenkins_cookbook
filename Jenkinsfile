def cookbook = 'test_jenkins_cookbook'

def stableBranch = 'master'
def currentBranch = env.BRANCH_NAME

def cookbookDirectory = "D:/cookbooks/${cookbook}"

def fetch(scm, cookbookDirectory, currentBranch){
  checkout([$class: 'GitSCM',
    branches: scm.branches,
    doGenerateSubmoduleConfigurations: scm.doGenerateSubmoduleConfigurations,
    extensions: scm.extensions + [
      [$class: 'RelativeTargetDirectory',relativeTargetDir: cookbookDirectory],
      [$class: 'CleanBeforeCheckout'],
      [$class: 'LocalBranch', localBranch: currentBranch]
    ],
    userRemoteConfigs: scm.userRemoteConfigs
  ])
}

stage('Linting') {
  node('windows') {

    echo "cookbook: ${cookbook}"
    echo "current branch: ${currentBranch}"
    echo "checkout directory: ${cookbookDirectory}"
    try{
      fetch(scm, cookbookDirectory, currentBranch)
      dir(cookbookDirectory){
        // clean out any old artifacts from the cookbook directory including the berksfile.lock file
        bat "del Berksfile.lock"
      }

     dir(cookbookDirectory) {
        try {
          bat "chef exec cookstyle ."
        }
      }
      currentBuild.result = 'SUCCESS'
    }
    catch(err){
      currentBuild.result = 'FAILED'
      notify_stash(building_pull_request)
      throw err
    }
  }
}

stage('Unit Testing'){
  node('windows') {
    try {
      fetch(scm, cookbookDirectory, currentBranch)
      dir(cookbookDirectory) {
        bat "berks install"
        bat "chef exec rspec ."
        currentBuild.result = 'SUCCESS'
      }
    }
    catch(err){
      currentBuild.result = 'FAILED'
      throw err
    }
  }
}

stage('Functional (Kitchen)') {
  node('kitchen') {
    try{
      fetch(scm, cookbookDirectory, currentBranch)
      dir(cookbookDirectory) {
       bat '''
          set KITCHEN_YAML=.kitchen.jenkins.yml
          kitchen verify
        '''
      }
      currentBuild.result = 'SUCCESS'
    }
    catch(err){
      currentBuild.result = 'FAILED'
    }
    finally {
      dir(cookbookDirectory) {
        bat '''
          set KITCHEN_YAML=.kitchen.jenkins.yml
          kitchen destroy
        '''
      }
    }
  }
}

stage('Publishing') {
  node('kitchen') {
    try{
      dir(cookbookDirectory) {
        echo "#TODO: Add tasks for publishing here."
        currentBuild.result = 'SUCCESS'
      }
    }
    catch(err){
      currentBuild.result = 'FAILED'
    }
  }
}

stage('Clean up') {
  node('kitchen') {
    try{
      dir(cookbookDirectory) {
        echo "#TODO: Add tasks for clean up here."
        currentBuild.result = 'SUCCESS'
      }
    }
    catch(err){
      currentBuild.result = 'FAILED'
    }
  }
}