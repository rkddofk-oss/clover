name: Convert markdown to PPTX and commit to main

on:
  push:
    paths:
      - 'christmas-in-meyongdong.md'
  workflow_dispatch: {}

jobs:
  convert-and-push:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
          persist-credentials: false

      - name: Install pandoc
        run: |
          sudo apt-get update
          sudo apt-get install -y pandoc

      - name: Convert markdown to PPTX
        run: |
          MD_FILE="christmas-in-meyongdong.md"
          OUT_FILE="christmas-in-meyongdong.pptx"
          if [ ! -f "$MD_FILE" ]; then
            echo "Error: $MD_FILE not found in repository root"
            exit 1
          fi
          pandoc "$MD_FILE" --slide-level=1 -o "$OUT_FILE"
          ls -lah "$OUT_FILE"

      - name: Commit and push generated PPTX to main (using PAT)
        env:
          GIT_AUTHOR_NAME: github-actions[bot]
          GIT_AUTHOR_EMAIL: github-actions[bot]@users.noreply.github.com
        run: |
          # Configure git author
          git config user.name "${GIT_AUTHOR_NAME}"
          git config user.email "${GIT_AUTHOR_EMAIL}"

          OUT_FILE="christmas-in-meyongdong.pptx"

          # Ensure we have latest main
          git fetch origin main
          git checkout main
          git reset --mixed origin/main

          # Stage the generated file
          git add "$OUT_FILE" || true

          # If no staged changes, exit
          if git diff --staged --quiet; then
            echo "No changes to commit"
            exit 0
          fi

          git commit -m "Add generated PPTX from christmas-in-meyongdong.md via pandoc"

          # Use PAT to push directly to main
          if [ -z "${{ secrets.ACTIONS_PAT }}" ]; then
            echo "ERROR: ACTIONS_PAT secret is not set"
            exit 1
          fi

          # Replace origin URL to include PAT for push (keeps token out of logs)
          git remote set-url origin https://x-access-token:${{ secrets.ACTIONS_PAT }}@github.com/${{ github.repository }}.git

          # Push to main
          git push origin main

      - name: Upload PPTX as workflow artifact
        uses: actions/upload-artifact@v4
        with:
          name: christmas-in-meyongdong-pptx
          path: christmas-in-meyongdong.pptx
