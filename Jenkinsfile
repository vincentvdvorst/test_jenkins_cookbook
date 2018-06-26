node {
  try {
    stage('Prepare') {
      deleteDir()
      checkout scm
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
      echo "Testing"
    }
  }
}