node {
  stage('JIRA') {
    withEnv(['JIRA_SITE=Bilbo']) {
      def testIssue = [fields: [ project: [key: 'BIL'],
                                 summary: 'New JIRA Created from Jenkins.',
                                 description: 'New JIRA Created from Jenkins.',
                                 issuetype: [name: 'Story']]]

      response = jiraNewIssue issue: testIssue

      echo response.successful.toString()
      echo response.data.toString()
    }
  }
}
