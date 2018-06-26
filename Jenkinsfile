node {
  environment {
    KITCHEN_YAML = '.kitchen.jenkins.yml'
  }
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
    stage('Chef Integration Testing') {
      echo "Starting Kitchen tests, this may take awhile."
      bat '''
        set KITCHEN_YAML=.kitchen.jenkins.yml
        kitchen verify
      '''
    }
    stage('publish') {
      echo "Testing"
    }
  } finally {
    stage('cleanup') {
      // deleteDir()
      bat "kitchen destroy"
      echo "Testing"
    }
  }
}