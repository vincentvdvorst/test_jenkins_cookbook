node {
  try {
    stage('Prepare') {
      checkout scm
    }
    stage('Chef Linting') {
      bat "chef exec cookstyle ."
    }
    stage('Chef Unit Testing') {
      bat "chef exec rspec"
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