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
    credentialId = "16fab210-1259-4cb5-9acc-c2134ac32ea4"
    version = cookbookPipeline.getNewVersion(scm, cookbookDirectory, currentBranch)
    dir(cookbookDirectory) {
      withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: credentialId, usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD']]) {
        echo "New version is: ${version}"
        version = "22.3.2"
        gitURL = GIT_URL.split("//")[1]
        encodedPassword = java.net.URLEncoder.encode(GIT_PASSWORD, "UTF-8")
        powershell(script: "git config user.name \"Jenkins Builder\"")
        powershell(script: "git config user.email \"cog@gamestop.com\"")
        powershell(script: "git tag -a ${version} -m ${version}")
        powershell(script: "git push https://'${GIT_USERNAME}':'${encodedPassword}'@${gitURL} ${newVer}")
      }
    }
  }
}