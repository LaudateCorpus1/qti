#!/usr/bin/env groovy

def gemfiles = [
    'nokogiri-1.6.gemfile',
    'nokogiri-1.8.gemfile',
    'rails-4.2.gemfile',
    'rails-5.0.gemfile',
    'rails-5.1.gemfile',
    'sanitize-4.2.gemfile',
    'sanitize-4.5.gemfile',
]
def buildMatrix = gemfiles.collectEntries { gemfile ->
    ['2.3', '2.4'].collectEntries { ruby ->
        ["Ruby ${ruby} - ${gemfile}": {
            sh """
                docker-compose run -e BUNDLE_GEMFILE="spec/gemfiles/${gemfile}" \
                    --name "${env.JOB_NAME.replaceAll("/", "-")}-${env.BUILD_ID}-ruby-${ruby}-${gemfile}-rspec" \
                    app /bin/bash -lc "rvm-exec ${ruby} bundle install --jobs 3 && rvm-exec ${ruby} bundle exec rspec"
            """
        }]
    }
}
buildMatrix << ['Lint': {
    sh 'docker-compose run --rm app /bin/bash -lc "rvm-exec 2.4 bundle exec rubocop --fail-level autocorrect"'
}]

pipeline {
    agent {
        label 'docker'
    }
    options {
        buildDiscarder(logRotator(numToKeepStr: '50'))
        timeout(time: 20, unit: 'MINUTES')
    }
    stages {
        stage('Build') {
            steps {
                sh 'docker-compose build --pull'
            }
        }
        stage('Test') {
            steps { script { parallel buildMatrix } }
            post {
                always {
                    sh "docker cp \"${env.JOB_NAME.replaceAll("/", "-")}-${env.BUILD_ID}-ruby-2.4-nokogiri-1.8.gemfile-rspec:/app/coverage\" ."
                    sh 'docker-compose down --remove-orphans --rmi all'
                    archiveArtifacts artifacts: 'coverage/*.json', fingerprint: true
                }
            }
        }
    }
}