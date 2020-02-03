pipeline {
  agent any

  stages {
    stage('Ticket Creation') {
      steps {
        script {
          withEnv(['JIRA_SITE=ohtest']) {
            def testIssue = [fields: [ project: [key: 'OHTEST'],
                                       summary: env.BRANCH_NAME,
                                       description: 'Jira CR created for ' + env.BRANCH_NAME,
                                       issuetype: [name: 'Story']]]

            response = jiraNewIssue issue: testIssue

            env.JIRA_TICKET = response.data.id

            echo env.JIRA_TICKET
          }
        }
      }
    }

    stage('Move ticket to in progress') {
      steps {
        script {
          withEnv(['JIRA_SITE=ohtest']) {
            // need to check on each jira project the transition ID - this is unique
            transitionTicket(transition_stages.in_progress)
          }
        }
      }
    }

    stage('Build etc..') {
      steps {
        script {
          // try to build, if fails it will catch the error
          try {
            sh 'false'
          } catch(e) {
            env.BUILD_ERROR_MSG = e
            // fail the catch, so the post step is correctly ran
            error(e)
          }
        }
      }
    }
  }

  post {
        success {
            script {
                // If the deployment succeeds, the CR will be closed with an appropriate message.
                echo "Build is successful, sending CR to complete"
                withEnv(['JIRA_SITE=ohtest']) {
                  transitionTicket(transition_stages.done)
                }
            }
        }
        failure {
            script {
                // If the deployment fails, the CR will be closed via the failure outcome.
                withEnv(['JIRA_SITE=ohtest']) {
                  //transitionTicket(transition_stages.failed)
                  commentTicket(env.BUILD_ERROR_MSG)
                }
            }
        }
    }
}

def transition_stages = [
    in_progress: 31,
    done: 41,
    failed: 44
]

def transitionTicket(transition_id) {
  def transitionInput =
  [
      transition: [
          id: transition_id
      ]
  ]

  jiraTransitionIssue idOrKey: env.JIRA_TICKET, input: transitionInput
}

def commentTicket(message) {
  def comment = [ body: message ]
  jiraAddComment idOrKey: env.JIRA_TICKET, input: comment
}
