pipeline {
  options {
    disableConcurrentBuilds()
  }

  agent {
    label "nodejs-app-slave"
  }

  parameters {
    string(name: 'COMMIT_ID', defaultValue: '', description: "The git commit id to indicate the codebase for building. By default it's the environment GIT_COMMIT from Jenkins")
    booleanParam(name: 'BUILD', defaultValue: true, description: "Build and publish Docker image to AWS ECR.")
    booleanParam(name: "DEPLOY_TO_KOBITON_TEST", defaultValue: false, description: "Deploy to Kobiton Test")
    booleanParam(name: "DEPLOY_TO_KOBITON_STAGING", defaultValue: false, description: "Deploy to Kobiton Staging")
    booleanParam(name: "DEPLOY_TO_KOBITON_PRODUCTION", defaultValue: false, description: "Deploy to Kobiton Production")
    booleanParam(name: "DEPLOY_TO_ING_PRODUCTION", defaultValue: false, description: "Deploy to Ing Production")
    booleanParam(name: "DEPLOY_TO_UBER_STAGING", defaultValue: false, description: "Deploy to Uber Staging")
    booleanParam(name: "DEPLOY_TO_UBER_PRODUCTION", defaultValue: false, description: "Deploy to Uber Production")
    string(name: 'DEPLOY_REPO_CODEBASE_ID', defaultValue: '', description: 'Either the remote branch ("origin/<branch name>"),  git commit id ("aaec22") or git tag (release-v3.6.0) of the env repo that is used to deploy the app. By default it is "origin/master"')
  }

  environment {
    CI_DEPLOY_APP_NAME = "documentation"
    CI_DOCKER_IMAGE_NAME = "documentation"
  }

  stages {
    stage('Set variables for CI') {
      steps {
        script {
          GIT_COMMIT_ID=sh(script: "[ -z $params.COMMIT_ID ] && echo ${env.GIT_COMMIT.take(6)} || git rev-parse --short=6 $params.COMMIT_ID", returnStdout: true).trim()
        }
      }
    }

    stage('Pull docker-ci tool') {
      when {
        expression {
          return !env.BRANCH_NAME.startsWith("PR-")
        }
      }

      steps {
        script {
          // login to AWS ECR
          sh('eval $(aws ecr get-login --no-include-email --region ap-southeast-1)')

          sh("docker pull 580359070243.dkr.ecr.ap-southeast-1.amazonaws.com/docker-ci:latest")

          // Alias it to call below statement
          sh("docker tag 580359070243.dkr.ecr.ap-southeast-1.amazonaws.com/docker-ci:latest docker-ci:latest")
        }
      }
    }

    stage('Build and publish to AWS ECR') {
      when {
        expression {
          return !env.BRANCH_NAME.startsWith("PR-") && params.BUILD
        }
      }

      steps {
        script {
          sh("docker run --rm \
                -v /var/run/docker.sock:/var/run/docker.sock \
                docker-ci:latest \
                  --git-url $env.GIT_URL \
                  --commit-id $GIT_COMMIT_ID \
                  build --docker --docker-image-name $env.CI_DOCKER_IMAGE_NAME \
                  --docker-image-tag $GIT_COMMIT_ID")
          sh("docker run --rm \
                -v /var/run/docker.sock:/var/run/docker.sock \
                docker-ci:latest archive --docker-registry \
                --docker-image-name $env.CI_DOCKER_IMAGE_NAME \
                --docker-image-tag $GIT_COMMIT_ID")
        }
      }
    }

    stage("Deploy to Kobiton Test") {
      when {
        expression {
          return !env.BRANCH_NAME.startsWith("PR-") && params.DEPLOY_TO_KOBITON_TEST
        }
      }

      steps {
        script {
          sh("docker run --rm \
            -v /var/run/docker.sock:/var/run/docker.sock \
            docker-ci:latest deploy \
            ${params.DEPLOY_REPO_CODEBASE_ID ? "--env-codebase-id $params.DEPLOY_REPO_CODEBASE_ID" : ''} \
            --env-name kobiton-test --app-name $env.CI_DEPLOY_APP_NAME \
            --docker-image-tag $GIT_COMMIT_ID")
        }
      }
    }

    stage("Deploy to Kobiton Staging") {
      when {
        expression {
          return params.DEPLOY_TO_KOBITON_STAGING
        }
      }

      steps {
        script {
          sh("docker run --rm \
            -v /var/run/docker.sock:/var/run/docker.sock \
            docker-ci:latest deploy \
            ${params.DEPLOY_REPO_CODEBASE_ID ? "--env-codebase-id $params.DEPLOY_REPO_CODEBASE_ID" : ''} \
            --env-name kobiton-staging --app-name $env.CI_DEPLOY_APP_NAME \
            --docker-image-tag $GIT_COMMIT_ID")
        }
      }
    }

    stage("Deploy to Kobiton Production") {
      when {
        expression {
          return params.DEPLOY_TO_KOBITON_PRODUCTION
        }
      }

      steps {
        script {
          sh("docker run --rm \
            -v /var/run/docker.sock:/var/run/docker.sock \
            docker-ci:latest deploy \
            ${params.DEPLOY_REPO_CODEBASE_ID ? "--env-codebase-id $params.DEPLOY_REPO_CODEBASE_ID" : ''} \
            --env-name kobiton-prod --app-name $env.CI_DEPLOY_APP_NAME \
            --docker-image-tag $GIT_COMMIT_ID")
        }
      }
    }

    stage("Deploy to Ing Production") {
      when {
        expression {
          return params.DEPLOY_TO_ING_PRODUCTION
        }
      }

      steps {
        script {
          sh("docker run --rm \
            -v /var/run/docker.sock:/var/run/docker.sock \
            docker-ci:latest deploy \
            ${params.DEPLOY_REPO_CODEBASE_ID ? "--env-codebase-id $params.DEPLOY_REPO_CODEBASE_ID" : ''} \
            --env-name ing-prod --app-name $env.CI_DEPLOY_APP_NAME \
            --docker-image-tag $GIT_COMMIT_ID")
        }
      }
    }

    stage("Deploy to Uber Staging") {
      when {
        expression {
          return params.DEPLOY_TO_UBER_STAGING
        }
      }

      steps {
        script {
          sh("docker run --rm \
            -v /var/run/docker.sock:/var/run/docker.sock \
            docker-ci:latest deploy \
            ${params.DEPLOY_REPO_CODEBASE_ID ? "--env-codebase-id $params.DEPLOY_REPO_CODEBASE_ID" : ''} \
            --env-name uber-staging --app-name $env.CI_DEPLOY_APP_NAME \
            --docker-image-tag $GIT_COMMIT_ID")
        }
      }
    }

    stage("Deploy to Uber Production") {
      when {
        expression {
          return params.DEPLOY_TO_UBER_PRODUCTION
        }
      }

      steps {
        script {
          sh("docker run --rm \
            -v /var/run/docker.sock:/var/run/docker.sock \
            docker-ci:latest deploy \
            ${params.DEPLOY_REPO_CODEBASE_ID ? "--env-codebase-id $params.DEPLOY_REPO_CODEBASE_ID" : ''} \
            --env-name uber-prod --app-name $env.CI_DEPLOY_APP_NAME \
            --docker-image-tag $GIT_COMMIT_ID")
        }
      }
    }

  }

  post {
    failure {
      script {
        sh("docker run --rm \
              docker-ci:latest notify \
              --failure \
              --job-param-name COMMIT_ID --job-param-value ${GIT_COMMIT_ID} \
              --job-param-name BUILD --job-param-value ${params.BUILD} \
              --job-param-name DEPLOY_TO_KOBITON_TEST --job-param-value ${params.DEPLOY_TO_KOBITON_TEST} \
              --job-param-name DEPLOY_TO_KOBITON_STAGING --job-param-value ${params.DEPLOY_TO_KOBITON_STAGING} \
              --job-param-name DEPLOY_TO_KOBITON_PRODUCTION --job-param-value ${params.DEPLOY_TO_KOBITON_PRODUCTION} \
              --job-param-name DEPLOY_TO_ING_PRODUCTION --job-param-value ${params.DEPLOY_TO_ING_PRODUCTION} \
              --job-param-name DEPLOY_TO_UBER_STAGING --job-param-value ${params.DEPLOY_TO_UBER_STAGING} \
              --job-param-name DEPLOY_TO_UBER_PRODUCTION --job-param-value ${params.DEPLOY_TO_UBER_PRODUCTION} \
              ${params.DEPLOY_REPO_CODEBASE_ID ? "--job-param-name DEPLOY_REPO_CODEBASE_ID --job-param-value $params.DEPLOY_REPO_CODEBASE_ID": ''} \
              --ci-job-url ${env.BUILD_URL} \
              --ci-job-id ${env.BUILD_ID} \
              --git-repo-name kobiton/documentation")
      }
    }

    success {
      script {
        sh("docker run --rm \
              docker-ci:latest notify \
              --job-param-name COMMIT_ID --job-param-value ${GIT_COMMIT_ID} \
              --job-param-name BUILD --job-param-value ${params.BUILD} \
              --job-param-name DEPLOY_TO_KOBITON_TEST --job-param-value ${params.DEPLOY_TO_KOBITON_TEST} \
              --job-param-name DEPLOY_TO_KOBITON_STAGING --job-param-value ${params.DEPLOY_TO_KOBITON_STAGING} \
              --job-param-name DEPLOY_TO_KOBITON_PRODUCTION --job-param-value ${params.DEPLOY_TO_KOBITON_PRODUCTION} \
              --job-param-name DEPLOY_TO_ING_PRODUCTION --job-param-value ${params.DEPLOY_TO_ING_PRODUCTION} \
              --job-param-name DEPLOY_TO_UBER_STAGING --job-param-value ${params.DEPLOY_TO_UBER_STAGING} \
              --job-param-name DEPLOY_TO_UBER_PRODUCTION --job-param-value ${params.DEPLOY_TO_UBER_PRODUCTION} \
              ${params.DEPLOY_REPO_CODEBASE_ID ? "--job-param-name DEPLOY_REPO_CODEBASE_ID --job-param-value $params.DEPLOY_REPO_CODEBASE_ID": ''} \
              --ci-job-url ${env.BUILD_URL} \
              --ci-job-id ${env.BUILD_ID} \
              --git-repo-name kobiton/documentation")
      }
    }
  }
}
