yaml
name: Create PR Workflow (Best Practices)

on:
  workflow_dispatch:
    inputs:
      branch_type:
        description: 'Type of change for branch name prefix'
        required: true
        type: choice
        options:
        - feat
        - fix
        - docs
        - refactor
        - chore
        default: 'feat'
      branch_name:
        description: 'Short description for branch name (e.g., user-authentication)'
        required: true
      files_to_add:
        description: 'Files/folders to stage (space-separated). Defaults to all changes.'
        required: false
        default: '.'
      commit_scope:
        description: 'Scope of the commit (e.g., auth, api). Optional.'
        required: false
      commit_description:
        description: 'Commit description (first line of commit message)'
        required: true
      commit_body:
        description: 'Optional long description for the commit body.'
        required: false
      pr_title:
        description: 'Title for the Pull Request. Defaults to commit description.'
        required: false

jobs:
  create_pr:
    runs-on: ubuntu-latest
    steps:
      - name: 1. Pre-flight | Check repository state
        uses: actions/checkout@v4
        with:
          fetch-depth: 0 # Fetches all history for all branches and tags

      - name: 2. Pre-flight | Display git status
        run: |
          echo "### Current Git Status"
          git status

      - name: 3. Pre-flight | Scan for secrets
        uses: gitleaks/gitleaks-action@v2
        with:
          # This will fail the workflow if secrets are found.
          # You can change this to `continue-on-error: true` and just have it report.
          fail: true

      - name: 4. Branch | Set up Git credentials
        run: |
          git config --global user.name "${{ github.actor }}"
          git config --global user.email "${{ github.actor }}@users.noreply.github.com"

      - name: 5. Branch | Create and switch to a new branch
        run: |
          BRANCH_NAME="${{ github.event.inputs.branch_type }}/${{ github.event.inputs.branch_name }}"
          echo "Creating branch: $BRANCH_NAME"
          if git rev-parse --verify --quiet "origin/$BRANCH_NAME"; then
            echo "::error::Remote branch origin/$BRANCH_NAME already exists."
            exit 1
          fi
          if git rev-parse --verify --quiet "$BRANCH_NAME"; then
            echo "::error::Local branch $BRANCH_NAME already exists."
            exit 1
          fi
          git checkout -b $BRANCH_NAME
          echo "BRANCH_NAME=$BRANCH_NAME" >> $GITHUB_ENV

      - name: 6. Staging | Stage specified files
        run: |
          echo "Staging files: ${{ github.event.inputs.files_to_add }}"
          git add ${{ github.event.inputs.files_to_add }}

      - name: 7. Staging | Verify staged changes
        run: |
          echo "### Staged Changes"
          git status

      - name: 8. Commit | Construct commit message and commit
        run: |
          COMMIT_TITLE="${{ github.event.inputs.branch_type }}${{ github.event.inputs.commit_scope && format('({0})', github.event.inputs.commit_scope) || '' }}: ${{ github.event.inputs.commit_description }}"
          
          # Handling husky/pre-commit hooks:
          # If you have client-side hooks (like husky), they won't run in this environment
          # unless you install dependencies (npm install) and the hooks are configured to run.
          # If a pre-commit hook fails, this step will fail. You can then check the logs.
          # For auto-fixing, you would need to add steps to run linters/formatters before commit.
          
          if [ -z "${{ github.event.inputs.commit_body }}" ]; then
            git commit -m "$COMMIT_TITLE"
          else
            git commit -m "$COMMIT_TITLE" -m "${{ github.event.inputs.commit_body }}"
          fi
          echo "COMMIT_TITLE=$COMMIT_TITLE" >> $GITHUB_ENV

      - name: 9. Push | Push changes to remote
        run: |
          # Pre-push hooks: Similar to pre-commit, they need the right environment to run.
          # If a pre-push hook fails (e.g., failing tests), this step will fail.
          # The user's guide mentions retrying after auto-fixes. This workflow
          # would need to be more complex to handle that logic automatically.
          # A simple approach is to fail, let the user see the error, and they can
          # fix it manually or adjust the workflow.
          git push -u origin ${{ env.BRANCH_NAME }}

      - name: 10. PR | Create Pull Request
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          PR_TITLE="${{ github.event.inputs.pr_title || env.COMMIT_TITLE }}"
          PR_BODY=$(cat <<'EOF'
          ## What changed
          - Brief summary of changes

          ## Why this change
          - Context and motivation

          ## Testing done
          - How changes were verified

          ## Breaking changes
          - Any breaking changes (if applicable)
          EOF
          )
          
          gh pr create \
            --base "main" \
            --title "$PR_TITLE" \
            --body "$PR_BODY"
            
      - name: 11. Verification | Show remaining unstaged changes
        run: |
          echo "### Remaining unstaged changes (if any)"
          git status

