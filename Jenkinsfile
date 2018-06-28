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


class SemVer {
  def major, minor, patch

  SemVer() {
    this.major = 0
    this.minor = 0
    this.patch = 0
  }

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

  def set(major, minor, patch) {
    try {
      this.major = major
      this.minor = minor
      this.patch = patch
    }
    catch(err) {
      throw new Exception("This method expects 3 integer values for major, minor and patch versions")
    }
  }

  def set(semverstr) {
    try {
      this.major = semverstr.split("\\.")[0].toInteger()
      this.minor = semverstr.split("\\.")[1].toInteger()
      this.patch = semverstr.split("\\.")[2].toInteger()
    }
    
    catch(err) {
      throw new Exception("This method expects a sane Semantic Version string: \"major.minor.patch\" e.g. \"2.1.1\"")
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

def versionPin(currentEnvironment, chefRepo, cookbookDirectory, cookbook, versionPinOperator ) {
  dir(chefRepo) {
    environments = bat(returnStdout: true, script: """
      @echo off
      knife environment list
    """).trim().split()

    if (environments.contains(currentEnvironment)) {
      println "Environment already exists on Chef server, downloading..."
      bat "knife download environments/${currentEnvironment}.json"
    } else {
      throw new Exception("Please ensure your target environment exists on the Chef server.")
    }

    def jsonData = readJSON file: "${chefRepo}/environments/${currentEnvironment}.json"

    version = new SemVer()

    def metadataLines = readFile "${cookbookDirectory}/metadata.rb"

    for (line in metadataLines.split("\n")) {
      if (line ==~ /^version.*/) {
        version.set(line.split(" ")[1].replace("\'", ""))
        println version.toString()
      }
    }

    if (jsonData.containsKey('cookbook_versions')){
      jsonData['cookbook_versions']["${cookbook}"] = versionPinOperator + " " + version.toString()
    } else {
      def cookbookVersionsMap = [:]
      jsonData['cookbook_versions'] = [:]
      jsonData['cookbook_versions']["${cookbook}"] = versionPinOperator + " " + version.toString()
    }
    
    readJSON file: "${chefRepo}/environments/${currentEnvironment}.json"
    writeJSON file: "${chefRepo}/environments/${currentEnvironment}.json", json: jsonData, pretty:2

    bat "knife environment from file ${chefRepo}/environments/${currentEnvironment}.json"

    currentBuild.result = 'SUCCESS'
  }
}

stage('Versioning') {
  node {
    try {
      echo "Checking if version is updated on stable branch."
      fetch(scm, cookbookDirectory, currentBranch)
      dir(cookbookDirectory) {

        newVersion = new SemVer()

        def metadataLines = readFile "metadata.rb"

        for (line in metadataLines.split("\n")) {
          if (line ==~ /^version.*/) {
            newVersion.set(line.split(" ")[1].replace("\'", ""))
          }
        }

        cookbookDetails = ""

        currentVersion = new SemVer()

        try {
          cookbookDetails = bat(returnStdout: true, script: """
            @echo off
            knife cookbook show ${cookbook}
          """)
          currentVersion.set(cookbookDetails.split()[1])
        }
        catch(err) {
          echo "Cookbook is not present on Chef server, no version bump is required."
        }

        if (!newVersion.isNewerThan(currentVersion)) {
          throw new Exception("The version that has been set is not newer than the previous version.")
        } else {
          echo "The version has been set appropriately. Existing version: ${currentVersion.toString()}, new version is: ${newVersion.toString()}"
        }
      }
      currentBuild.result = 'SUCCESS'
    }
    catch(err) {
      currentBuild.result = 'FAILED'
      error err.getMessage()
      throw err
    }
  }
}

stage('Linting') {
  node {

    echo "cookbook: ${cookbook}"
    echo "current branch: ${currentBranch}"
    echo "checkout directory: ${cookbookDirectory}"
    try {
      fetch(scm, cookbookDirectory, currentBranch)
      dir(cookbookDirectory){
        // Delete berksfile.lock file
        bat "del Berksfile.lock"
      }

      dir(cookbookDirectory) {
        bat "chef exec cookstyle ."
      }
      currentBuild.result = 'SUCCESS'
    }
    catch(err) {
      currentBuild.result = 'FAILED'
      throw err
    }
  }
}

stage('Unit Testing') {
  node {
    try {
      fetch(scm, cookbookDirectory, currentBranch)
      dir(cookbookDirectory) {
        bat "berks install"
        bat "chef exec rspec ."
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

stage('Publishing') {
  node {
    if ( currentBranch == stableBranch ) {
      echo "Attempting upload of stable branch cookbook to Chef server."
      try{
        fetch(scm, cookbookDirectory, currentBranch)
        dir(cookbookDirectory) {
          bat "berks update"
          bat "berks vendor"
          bat "berks upload --halt-on-frozen"
          currentBuild.result = 'SUCCESS'
        }
      }
      catch(err){
        currentBuild.result = 'FAILED'
        throw err
      }
    } else {
      echo "Skipping Publishing stage as this is not the stable branch."
    }
  }
}

stage('Pinning in QA') {
  if (currentBranch == currentBranch) {
    def approval = input(message: "Deploy to ${qaEnvironment}?", ok: 'Yes', 
                          parameters: [booleanParam(defaultValue: true, 
                          description: 'Update version pins in environment?',name: 'Yes?')])

    if (approval == true) {
      node {
        try{
          fetch(scm, cookbookDirectory, currentBranch)
          versionPin(qaEnvironment, chefRepo, cookbookDirectory, cookbook, versionPinOperator)
        }
        catch(err){
          println err.getMessage()
          currentBuild.result = 'FAILED'
          throw err
        }
      }
    }
  } else {
    echo "Skipping Pinning stage"
  }
}

stage('Pinning in Prod') {
  if (currentBranch == currentBranch) {
    def approval = input(message: "Deploy to ${prodEnvironment}?", ok: 'Yes', 
                          parameters: [booleanParam(defaultValue: true, 
                          description: 'Update version pins in environment?',name: 'Yes?')])

    if (approval == true) {
      node {
        try{
          fetch(scm, cookbookDirectory, currentBranch)
          versionPin(prodEnvironment, chefRepo, cookbookDirectory, cookbook, versionPinOperator)
        }
        catch(err){
          println err.getMessage()
          currentBuild.result = 'FAILED'
          throw err
        }
      }
    }
  } else {
    echo "Skipping Pinning stage"
  }
}

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