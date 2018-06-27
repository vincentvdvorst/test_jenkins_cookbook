def cookbook = 'test_jenkins_cookbook'

def stableBranch = 'master'
def currentBranch = env.BRANCH_NAME

def VERSION_BUMP_REQUIRED = [
  "Berksfile",
  "Berksfile.lock",
  "Policyfile.rb",
  "Policyfile.lock.json",
  "recipes/.*",
  "attributes/.*",
  "libraries/.*",
  "files/.*",
  "templates/."
]

def cookbookDirectory = "D:/cookbooks/${cookbook}"

def fetch(scm, cookbookDirectory, currentBranch) {
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

class SemVer {
  def major, minor, patch

  SemVer(semverstr) {
    this.major = semverstr.split("\\.")[0]
    this.minor = semverstr.split("\\.")[1]
    this.patch = semverstr.split("\\.")[2]
  }

  def isGreaterThan(other) {
    if ((this.major > other.major) || (this.major == other.major && this.minor > other.minor) || (this.major == other.major && this.minor == other.minor && this.patch > other.patch)) {
      return true
    }
    return false
  }
}

// stage('Linting') {
//   node {

//     echo "cookbook: ${cookbook}"
//     echo "current branch: ${currentBranch}"
//     echo "checkout directory: ${cookbookDirectory}"
//     try {
//       fetch(scm, cookbookDirectory, currentBranch)
//       dir(cookbookDirectory){
//         // clean out any old artifacts from the cookbook directory including the berksfile.lock file
//         bat "del Berksfile.lock"
//       }

//       dir(cookbookDirectory) {
//         bat "chef exec cookstyle ."
//       }
//       currentBuild.result = 'SUCCESS'
//     }
//     catch(err) {
//       currentBuild.result = 'FAILED'
//       throw err
//     }
//   }
// }

// stage('Unit Testing') {
//   node {
//     try {
//       fetch(scm, cookbookDirectory, currentBranch)
//       dir(cookbookDirectory) {
//         bat "berks install"
//         bat "chef exec rspec ."
//       }
//       currentBuild.result = 'SUCCESS'
//     }
//     catch(err) {
//       currentBuild.result = 'FAILED'
//       throw err
//     }
//   }
// }

stage('Versioning') {
  node {
    try {
      fetch(scm, cookbookDirectory, currentBranch)
      dir(cookbookDirectory) {
        changed_files = bat(returnStdout: true, script: """
            @echo off
            git diff --name-only master
          """
        ).trim().split()
        println VERSION_BUMP_REQUIRED
        println changed_files
        if (changed_files.contains('metadata.rb')) {
          metadata_lines = bat(returnStdout: true, script: "git diff --unified=0 --no-color master:metadata.rb metadata.rb").split('\n')
          old_version = ""
          new_version = ""
          for (line in metadata_lines) {
            if (line ==~ /^(\+|\-)version.*/) {
              if (line ==~ /^\-version.*/) {
                old_version = line.split(" ")[1].replace("\'", "")
              }
              if (line ==~ /^\+version.*/) {
                new_version = line.split(" ")[1].replace("\'", "")
              }
            }
          }
          oldSemVer = new SemVer(old_version)
          newSemVer = new SemVer(new_version)
          println "Old version: ${old_version.split("\\.")}"
          println "New version: ${new_version.split("\\.")}"
          test1 = new SemVer('0.1.0')
          test2 = new SemVer('2.0.0')
          test3 = new SemVer('0.2.0')
          test4 = new SemVer('0.12.0')

          println "false, false, false"

          println test1.isGreaterThan(test2)
          println test1.isGreaterThan(test3)
          println test1.isGreaterThan(test4)

          println "true, true, true"

          println test2.isGreaterThan(test1)
          println test2.isGreaterThan(test3)
          println test2.isGreaterThan(test4)

          println "true, false, false"

          println test3.isGreaterThan(test1)
          println test3.isGreaterThan(test2)
          println test3.isGreaterThan(test4)

          println "true, false, true"

          println test4.isGreaterThan(test1)
          println test4.isGreaterThan(test2)
          println test4.isGreaterThan(test3)

          println oldSemVer.isGreaterThan(newSemVer)
        }
      }
      currentBuild.result = 'SUCCESS'
    }
    catch(err) {
      currentBuild.result = 'FAILED'
      throw err
    }
  }
}

// stage('Functional (Kitchen)') {
//   node {
//     try {
//       fetch(scm, cookbookDirectory, currentBranch)
//       dir(cookbookDirectory) {
//        bat '''
//           set KITCHEN_YAML=.kitchen.jenkins.yml
//           set KITCHEN_EC2_SSH_KEY_PATH=D:/kitchen/vvdvorst-us-east-1-sandbox.pem
//           kitchen verify
//         '''
//       }
//       currentBuild.result = 'SUCCESS'
//     }
//     catch(err) {
//       currentBuild.result = 'FAILED'
//     }
//     finally {
//       dir(cookbookDirectory) {
//         bat '''
//           set KITCHEN_YAML=.kitchen.jenkins.yml
//           kitchen destroy
//         '''
//       }
//     }
//   }
// }

stage('Publishing') {
  node {
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
  node {
    try {
      dir(cookbookDirectory) {
        echo "#TODO: Add tasks for clean up here."
        currentBuild.result = 'SUCCESS'
      }
    }
    catch(err) {
      currentBuild.result = 'FAILED'
    }
  }
}