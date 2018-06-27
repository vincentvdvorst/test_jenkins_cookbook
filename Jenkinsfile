def cookbook = 'test_jenkins_cookbook'

def stableBranch = 'master'
def currentBranch = env.BRANCH_NAME

def VERSION_BUMP_REQUIRED = [
  "Berksfile",
  "Berksfile.lock",
  "Policyfile.rb",
  "Policyfile.lock.json"
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
    this.major = semverstr.split("\\.")[0].toInteger()
    this.minor = semverstr.split("\\.")[1].toInteger()
    this.patch = semverstr.split("\\.")[2].toInteger()
  }

  def isNewerThan(other) {
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

        version_has_been_bumped = false
        version_bump_required = false

        // def VERSION_BUMP_REQUIRED = [
        //   "Berksfile",
        //   "Berksfile.lock",
        //   "Policyfile.rb",
        //   "Policyfile.lock.json",
        //   "recipes/.*",
        //   "attributes/.*",
        //   "libraries/.*",
        //   "files/.*",
        //   "templates/."
        // ]

        for (file in changed_files) {
          if ( file ==~ /files\/.*/ || file ==~ /recipes\/.*/ || file ==~ /attributes\/.*/ || file ==~ /libraries\/.*/ || file ==~ /templates\/.*/) {
            version_bump_required = true
          } else if ( VERSION_BUMP_REQUIRED.contains(file)) {
            println file
            version_bump_required = true
          }
        }

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

          if (!newSemVer.isNewerThan(oldSemVer)) {
            throw new Exception("The version that has been set is not newer than the previous version.")
          } else {
            version_has_been_bumped = true
          }
        }
      }
      println "################"
      println version_has_been_bumped
      println "################"
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