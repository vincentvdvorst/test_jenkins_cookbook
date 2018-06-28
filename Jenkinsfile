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

def fetch(scm, cookbookDirectory, currentBranch) {
  checkout([$class: 'GitSCM',
    branches: scm.branches,
    doGenerateSubmoduleConfigurations: scm.doGenerateSubmoduleConfigurations,
    extensions: scm.extensions + [
      [$class: 'RelativeTargetDirectory',relativeTargetDir: cookbookDirectory],
      [$class: 'CleanBeforeCheckout'],
      [$class: 'WipeWorkspace'],
      [$class: 'LocalBranch', localBranch: currentBranch]
    ],
    userRemoteConfigs: scm.userRemoteConfigs
  ])
}

class SemVer {
  def major, minor, patch

  SemVer(semverstr) {
    try {
      this.major = semverstr.split("\\.")[0].toInteger()
      this.minor = semverstr.split("\\.")[1].toInteger()
      this.patch = semverstr.split("\\.")[2].toInteger()
    }
    
    catch(err) {
      throw new Exception("This constructor expects a sane Semantic Version string: \"major.minor.patch\" e.g. \"2.1.1\"")
    }
  }

  SemVer(major, minor, patch) {
    try {
      this.major = major
      this.minor = minor
      this.patch = patch
    }
    catch(err) {
      throw new Exception("This constructor expects 3 integer values for major, minor and patch versions")
    }
  }

  def isNewerThan(other) {
    if ((this.major > other.major) || (this.major == other.major && this.minor > other.minor) || (this.major == other.major && this.minor == other.minor && this.patch > other.patch)) {
      return true
    }
    return false
  }

  String toString() {
    return "${this.major.toString()}.${this.minor.toString()}.${this.patch.toString()}"
  }
}

stage('Versioning') {
  node {
    try {
      fetch(scm, cookbookDirectory, currentBranch)
      dir(cookbookDirectory) {
      }
    //   fetch(scm, cookbookDirectory, stableBranch)
    //   fetch(scm, cookbookDirectory, currentBranch)
    //   dir(cookbookDirectory) {
    //     changed_files = bat(returnStdout: true, script: """
    //         @echo off
    //         git diff --name-only master
    //       """
    //     ).trim().split()

    //     version_has_been_bumped = false
    //     version_bump_required = false

    //     println changed_files.join(" ")

    //     for (file in changed_files) {
    //       if ( file ==~ /files\/.*/ || file ==~ /recipes\/.*/ || file ==~ /attributes\/.*/ || file ==~ /libraries\/.*/ || file ==~ /templates\/.*/) {
    //         version_bump_required = true
    //       } else if ( VERSION_BUMP_REQUIRED.contains(file)) {
    //         println file
    //         version_bump_required = true
    //       }
    //     }

    //     if (changed_files.contains('metadata.rb')) {
    //       metadata_lines = bat(returnStdout: true, script: "git diff --unified=0 --no-color master:metadata.rb metadata.rb").split('\n')
    //       old_version = "0.0.0"
    //       new_version = "0.0.0"
    //       for (line in metadata_lines) {
    //         if (line ==~ /^(\+|\-)version.*/) {
    //           if (line ==~ /^\-version.*/) {
    //             old_version = line.split(" ")[1].replace("\'", "")
    //           }
    //           if (line ==~ /^\+version.*/) {
    //             new_version = line.split(" ")[1].replace("\'", "")
    //           }
    //         }
    //       }
    //       oldSemVer = new SemVer(old_version)
    //       newSemVer = new SemVer(new_version)

    //       if (!newSemVer.isNewerThan(oldSemVer)) {
    //         throw new Exception("The version that has been set is not newer than the previous version.")
    //       } else {
    //         version_has_been_bumped = true
    //       }
    //     }
    //   }
      
    //   if (version_bump_required && !version_has_been_bumped) {
    //     throw new Exception("Changes have been made that require a version update.")
    //   }
    //   currentBuild.result = 'SUCCESS'
    }
    catch(err) {
      currentBuild.result = 'FAILED'
      error err.getMessage()
      throw err
    }
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
//       throw err
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

// stage('Publishing') {
//   node {
//     if ( currentBranch == stableBranch ) {
//       try{
//         dir(cookbookDirectory) {
//           bat "berks vendor"
//           bat "berks upload --halt-on-frozen"
//           currentBuild.result = 'SUCCESS'
//         }
//       }
//       catch(err){
//         currentBuild.result = 'FAILED'
//         throw err
//       }
//     } else {
//       echo "Skipping Publishing stage"
//     }
//   }
// }

// stage('Pinning in QA') {
//   if (currentBranch == stableBranch) {
//     def approval = input(message: "Deploy to ${qaEnvironment}?", ok: 'Yes', 
//                           parameters: [booleanParam(defaultValue: true, 
//                           description: 'Update version pins in environment?',name: 'Yes?')])

//     if (approval == true) {
//       node {
//         try{
//           dir(chefRepo) {
//             environments = bat(returnStdout: true, script: """
//               @echo off
//               knife environment list
//             """).trim().split()

//             if (environments.contains(qaEnvironment)) {
//               println "Environment already exists on Chef server, downloading..."
//               bat "knife download environments/${qaEnvironment}.json"
//             } else {
//               throw new Exception("Please ensure your target environment exists on the Chef server.")
//             }

//             def jsonData = readJSON file: "${chefRepo}/environments/${qaEnvironment}.json"

//             version = new SemVer('0.0.0')

//             def metadata_lines = readFile "${cookbookDirectory}/metadata.rb"

//             for (line in metadata_lines.split("\n")) {
//               if (line ==~ /^version.*/) {
//                 version = new SemVer(line.split(" ")[1].replace("\'", ""))
//                 println version.toString()
//               }
//             }

//             if (jsonData.containsKey('cookbook_versions')){
//               jsonData['cookbook_versions']["${cookbook}"] = versionPinOperator + " " + version.toString()
//             } else {
//               def cookbookVersionsMap = [:]
//               jsonData['cookbook_versions'] = [:]
//               jsonData['cookbook_versions']["${cookbook}"] = versionPinOperator + " " + version.toString()
//             }
            
//             readJSON file: "${chefRepo}/environments/${qaEnvironment}.json"
//             writeJSON file: "${chefRepo}/environments/${qaEnvironment}.json", json: jsonData, pretty:2

//             bat "knife environment from file ${chefRepo}/environments/${qaEnvironment}.json"

//             currentBuild.result = 'SUCCESS'
//           }
//         }
//         catch(err){
//           println err.getMessage()
//           currentBuild.result = 'FAILED'
//           throw err
//         }
//       }
//     }
//   } else {
//     echo "Skipping Pinning stage"
//   }
// }

// stage('Pinning in Prod') {
//   if (currentBranch == stableBranch) {
//     def approval = input(message: "Deploy to ${prodEnvironment}?", ok: 'Yes', 
//                           parameters: [booleanParam(defaultValue: true, 
//                           description: 'Update version pins in environment?',name: 'Yes?')])

//     if (approval == true) {
//       node {
//         try{
//           dir(chefRepo) {
//             environments = bat(returnStdout: true, script: """
//               @echo off
//               knife environment list
//             """).trim().split()

//             if (environments.contains(prodEnvironment)) {
//               println "Environment already exists on Chef server, downloading..."
//               bat "knife download environments/${prodEnvironment}.json"
//             } else {
//               throw new Exception("Please ensure your target environment exists on the Chef server.")
//             }

//             def jsonData = readJSON file: "${chefRepo}/environments/${prodEnvironment}.json"

//             version = new SemVer('0.0.0')

//             def metadata_lines = readFile "${cookbookDirectory}/metadata.rb"

//             for (line in metadata_lines.split("\n")) {
//               if (line ==~ /^version.*/) {
//                 version = new SemVer(line.split(" ")[1].replace("\'", ""))
//                 println version.toString()
//               }
//             }

//             if (jsonData.containsKey('cookbook_versions')){
//               jsonData['cookbook_versions']["${cookbook}"] = versionPinOperator + " " + version.toString()
//             } else {
//               def cookbookVersionsMap = [:]
//               jsonData['cookbook_versions'] = [:]
//               jsonData['cookbook_versions']["${cookbook}"] = versionPinOperator + " " + version.toString()
//             }
            
//             readJSON file: "${chefRepo}/environments/${prodEnvironment}.json"
//             writeJSON file: "${chefRepo}/environments/${prodEnvironment}.json", json: jsonData, pretty:2

//             bat "knife environment from file ${chefRepo}/environments/${prodEnvironment}.json"

//             currentBuild.result = 'SUCCESS'
//           }
//         }
//         catch(err){
//           println err.getMessage()
//           currentBuild.result = 'FAILED'
//           throw err
//         }
//       }
//     }
//   } else {
//     echo "Skipping Pinning stage"
//   }
// }

// stage('Clean up') {
//   node {
//     try {
//       dir(cookbookDirectory) {
//         bat '''
//           set KITCHEN_YAML=.kitchen.jenkins.yml
//           kitchen destroy
//         '''
//         currentBuild.result = 'SUCCESS'
//       }
//     }
//     catch(err) {
//       currentBuild.result = 'FAILED'
//       throw err
//     }
//   }
// }