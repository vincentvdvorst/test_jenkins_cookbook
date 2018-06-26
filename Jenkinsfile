def currentBranch = env.BRANCH_NAME


String determineRepoName() {
    return scm.getUserRemoteConfigs()[0].getUrl().tokenize('/')[3].split("\\.")[0]
}

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

node {

  def cookbook = determineRepoName
  def cookbookDirectory = "D:/chef/cookbooks/${cookbook}"

  try {
    stage('Prepare') {
      bat "rmdir /S /Q cookbookDirectory"
      bat "git clean -fdx"
      fetch(scm, cookbookDirectory, currentBranch)
    }
    stage('Chef Linting') {
      dir(cookbookDirectory){
        bat "chef exec cookstyle ."
      }
    }
    stage('test') {
      echo "Testing"
    }
    stage('package') {
      echo "Testing"
    }
    stage('publish') {
      echo "Testing"
    }
  } finally {
    stage('cleanup') {
      echo "Testing"
    }
  }
}