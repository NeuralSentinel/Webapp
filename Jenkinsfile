pipeline {
    agent any

    environment {
        GITHUB_TOKEN = credentials('github-token')  // GitHub token stored in Jenkins credentials
        OPENAI_API_KEY = credentials('openai-api-key')  // OpenAI API key
        OPENAI_ENDPOINT = 'https://api.openai.com/v1/chat/completions'  // OpenAI API endpoint
        GITHUB_API_URL = 'https://api.github.com'
        REPO_NAME = 'owner/repo'  // Replace with your GitHub repository path
    }

    stages {
        stage('Fetch Code Changes') {
            steps {
                script {
                    // Extract the Pull Request number
                    def prNumber = env.CHANGE_ID
                    if (!prNumber) {
                        error "This pipeline only works for pull requests!"
                    }
                    echo "Pull Request Number: #${prNumber}"

                    // Fetch PR code changes using GitHub API
                    sh """
                        curl -s -H "Authorization: token ${GITHUB_TOKEN}" \\
                        ${GITHUB_API_URL}/repos/${REPO_NAME}/pulls/${prNumber}/files \\
                        | jq -r '.[] | "\\(.filename):\\n\\(.patch)"' > pr_changes.txt
                    """
                    
                    echo "Code changes fetched and saved to pr_changes.txt"
                }
            }
        }

        stage('Send Code Changes to OpenAI') {
            steps {
                script {
                    // Read the code changes
                    def codeChanges = readFile('pr_changes.txt').trim()
                    echo "Code Changes:\n${codeChanges.take(500)}..." // Log first 500 chars for debugging

                    if (codeChanges) {
                        // Send code changes to OpenAI for review
                        def openaiResponse = sh(script: """
                            curl -s -X POST ${OPENAI_ENDPOINT} \\
                            -H "Content-Type: application/json" \\
                            -H "Authorization: Bearer ${OPENAI_API_KEY}" \\
                            -d '{
                                "model": "gpt-4",
                                "messages": [
                                    {"role": "system", "content": "You are a helpful AI assistant performing code reviews. Provide comments on improvements, bugs, and best practices."},
                                    {"role": "user", "content": "${codeChanges}"}
                                ]
                            }' | jq -r '.choices[0].message.content'
                        """, returnStdout: true).trim()

                        echo "OpenAI Response:\n${openaiResponse}"

                        // Save response to file
                        writeFile file: 'openai_review.txt', text: openaiResponse
                    } else {
                        error "No code changes detected!"
                    }
                }
            }
        }

        stage('Post Review Comments to Pull Request') {
            steps {
                script {
                    def prNumber = env.CHANGE_ID
                    def openaiReview = readFile('openai_review.txt').trim()

                    echo "Posting OpenAI Review to PR #${prNumber}..."

                    sh """
                        curl -s -X POST -H "Authorization: token ${GITHUB_TOKEN}" \\
                        -H "Accept: application/vnd.github.v3+json" \\
                        ${GITHUB_API_URL}/repos/${REPO_NAME}/issues/${prNumber}/comments \\
                        -d '{ "body": "${openaiReview.replace("\n", "\\n")}" }'
                    """
                    echo "Review posted successfully."
                }
            }
        }
    }
}

