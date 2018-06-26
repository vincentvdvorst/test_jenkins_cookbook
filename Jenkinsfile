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
      def workspace = pwd()
      echo "${workspace}"
      echo "Starting Kitchen tests, this may take awhile."
      bat """
        set KITCHEN_YAML=${workspace}/.kitchen.jenkins.yml
        kitchen verify
      """
    }
    stage('publish') {
      echo "Testing"
    }
  } finally {
    stage('cleanup') {
      // deleteDir()
      bat '''
        set KITCHEN_YAML=.kitchen.jenkins.yml
        kitchen destroy
      '''
      echo "Testing"
    }
  }
}