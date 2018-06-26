def currentBranch = env.BRANCH_NAME

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

  def chefrepo = "D:/chef/cookbooks"
  def cookbook = 'test_jenkins_cookbook'
  def cookbookDirectory = "${chefrepo}/${cookbook}"

  try {
    stage('Prepare') {
      dir(chefrepo){
        bat "rmdir /S /Q ${cookbook}"
      }
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