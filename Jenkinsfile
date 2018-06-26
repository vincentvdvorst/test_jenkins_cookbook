node {
  try {
    stage('Prepare') {
      checkout scm
      bat "git config core.autocrlf false"
      bat "git config core.eol lf"
      bat "git rm --cached -r ."
      bat "git reset --hard"
    }
    stage('Chef Linting') {
      bat "chef exec cookstyle ."
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
      deleteDir()
      echo "Testing"
    }
  }
}