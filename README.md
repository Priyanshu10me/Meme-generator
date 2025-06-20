# Meme-generator
 a simple way of creating a meme or emoji showing emotions with the help of text
emotion-emoji-app/
  ├── public/
  ├── src/
  │   ├── App.js
  │   ├── emotionMap.js
  │   └── index.js
  ├── package.json
  messages:
  - role: system
    content: >-
      You are a summarizer of GitHub issues. Emphasize key technical points or
      important questions.
  - role: user
    content: 'Summarize this issue, please - {{input}}'
model: gpt-4o
modelParameters:
  max_tokens: 4096
name: Summarize New Issue
on:
  issues:
    types: [opened]
permissions:
  issues: write
  contents: read
  models: read
jobs:
  summarize_issue:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
      - name: Install gh-models extension
        run: gh extension install https://github.com/github/gh-models
        env:
          GH_TOKEN: ${{ github.token }}
      - name: Create issue body file
        run: |
          cat > issue_body.txt << 'EOT'
          ${{ github.event.issue.body }}
          EOT
      - name: Summarize new issue
        run: |
          cat issue_body.txt | gh models run --file summarize.prompt.yml > summary.txt
        env:
          GH_TOKEN: ${{ github.token }}
      - name: Update issue with summary
        run: |
          SUMMARY=$(cat summary.txt)
          gh issue comment ${{ github.event.issue.number }} --body "### Issue Summary
          ${SUMMARY}"
        env:
          GH_TOKEN: ${{ github.token }}
        
