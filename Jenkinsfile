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

@Library('cookbookPipeline') _

// stage('Versioning') {
//   node {
//     cookbookPipeline.versionCheck(scm, cookbookDirectory, currentBranch, cookbook)
//   }
// }

// stage('Linting') {
//   node {
//     cookbookPipeline.lintTest(scm, cookbookDirectory, currentBranch, cookbook)
//   }
// }

// stage('Unit Testing') {
//   node {
//     cookbookPipeline.unitTests(scm, cookbookDirectory, currentBranch, cookbook)
//   }
// }

// stage('Functional (Kitchen)') {
//   node {
//     if (currentBranch == stableBranch) {
//       cookbookPipeline.functionalTests(scm, cookbookDirectory, currentBranch, cookbook)
//     } else {
//       echo "Skipping functional tests for branch: ${currentBranch}"
//     }
//   }
// }

stage('Publishing') {
  node {
    // cookbookPipeline.publish(scm, cookbookDirectory, currentBranch, stableBranch, cookbook)
    version = cookbookPipeline.getNewVersion(scm, cookbookDirectory, currentBranch)
    echo "New version is: ${version}"
  }
}