def cookbook = 'test_jenkins_cookbook'

def stableBranch = 'master'
def currentBranch = env.BRANCH_NAME

def qaEnvironment = 'qa'
def prodEnvironment = 'prod'

def versionPinOperator = "<="

def VERSION_BUMP_REQUIRED = [
  "Berksfile",
  "Berksfile.lock",
  "Policyfile.rb",
  "Policyfile.lock.json",
  "metadata.rb"
]

def chefRepo = "D:/chef-repo"
def chefRepoCookbookDirectory = "${chefRepo}/cookbooks"
def cookbookDirectory = "${chefRepoCookbookDirectory}/${cookbook}"

stage('Versioning') {
  node {
    versionCheck(scm, cookbookDirectory, currentBranch, cookbook)
  }
}

stage('Linting') {
  node {
    lintTest(scm, cookbookDirectory, currentBranch, cookbook)
  }
}

stage('Unit Testing') {
  node {
    unitTests(scm, cookbookDirectory, currentBranch, cookbook)
  }
}

stage('Functional (Kitchen)') {
  node {
    if (currentBranch == stableBranch) {
      functionalTests(scm, cookbookDirectory, currentBranch, cookbook)
    } else {
      echo "Skipping functional tests for branch: ${currentBranch}"
    }
  }
}
