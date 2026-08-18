Git Pre-Commit Secrets Detector

BashLicense: MIT

A lightweight, zero-dependency Bash hook to stop accidental commits of API keys, tokens, and passwords.
Features

    Runs automatically on git commit
    Scans only newly added or modified lines (not the whole file)
    Detects common secret patterns (AWS, GitHub, GitLab, Stripe, Slack, Google, private keys, etc.)
    Separate case‑sensitive and case‑insensitive matching
    Handles renamed files and filenames starting with dashes
    Lightweight Bash script – no dependencies beyond git and grep
    Easy to customize by editing pattern arrays

How It Works

    You stage files with git add
    You run git commit
    The hook extracts the added/modified lines from the staged diff
    Each line is checked against a set of regex patterns
    If a match is found, the commit is aborted and a report is shown

Installation
Option 1: Per repository

Copy the pre-commit file into your repository’s .git/hooks/ directory and make it executable:

cp pre-commit .git/hooks/pre-commitchmod +x .git/hooks/pre-commit

Option 2: Globally (all repositories)

Set a global hooks path and place the script there:
bash
 
  
 
 
mkdir -p ~/.git-hooks
cp pre-commit ~/.git-hooks/pre-commit
chmod +x ~/.git-hooks/pre-commit
git config --global core.hooksPath ~/.git-hooks
 
 

     

    Note: Global hooks apply to every repository. Make sure the patterns are suitable for your workflow.

Usage

After installation, simply commit as usual:
bash
 
  
 
 
git add .
git commit -m "Your message"
 
 

If a secret is found, the commit will be blocked with output similar to:
text
 
  
 
 
🔍 Scanning staged files for secrets...
❌ Potential secret detected in config.js matching pattern: ghp_[0-9a-zA-Z]{36}
🚫 Commit blocked. Remove secrets before committing, or use 'git commit --no-verify' to bypass.
 
 
Full Pre-Commit Script

If you prefer to copy the hook directly without downloading the file, use the script below. Save it as pre-commit, then make it executable with chmod +x pre-commit.
bash
 
  
 
 
#!/bin/bash
# Git pre-commit hook to detect secrets in staged changes
# Improved version: correct array iteration, robust diff extraction,
# separate case-sensitive and case-insensitive patterns, and safer handling.

# Case-sensitive patterns (tokens that are case-sensitive)
PATTERNS=(
  'AKIA[0-9A-Z]{16}'
  'sk_live_[0-9a-zA-Z]{24}'
  'ghp_[0-9a-zA-Z]{36}'
  'gho_[0-9a-zA-Z]{36}'
  'ghu_[0-9a-zA-Z]{36}'
  'ghs_[0-9a-zA-Z]{36}'
  'glpat-[0-9a-zA-Z]{20}'
  'xox[baprs]-[0-9a-zA-Z]{10,}'
  'AIza[0-9A-Za-z\-_]{35}'
  '-----BEGIN [A-Z]+ PRIVATE KEY-----'
)

# Case-insensitive patterns (generic assignments, etc.)
PATTERNS_CI=(
  '(password|passwd|pwd|secret)[[:space:]]*[=:][[:space:]]*[^[:space:]]+'
  '(api[_-]?key|apikey|access[_-]?token|auth[_-]?token)[[:space:]]*[=:][[:space:]]*[^[:space:]]+'
)

echo "🔍 Scanning staged files for secrets..."

# Get list of staged files (including renames, excluding deletions)
STAGED_FILES=$(git diff --cached --name-only --diff-filter=ACMR)

if [ -z "$STAGED_FILES" ]; then
    echo "✅ No staged files. All good."
    exit 0
fi

FOUND_SECRET=0

while IFS= read -r file; do
    [ -z "$file" ] && continue

    # Process each added line (exclude diff headers and file headers)
    while IFS= read -r line; do
        [ -z "$line" ] && continue
        line_matched=0

        # Check case-sensitive patterns
        for pattern in "${PATTERNS[@]}"; do
            if printf '%s' "$line" | grep -Eq "$pattern"; then
                echo "❌ Potential secret detected in $file matching pattern: $pattern"
                FOUND_SECRET=1
                line_matched=1
                break
            fi
        done

        # Check case-insensitive patterns only if no case-sensitive match
        if [ "$line_matched" -eq 0 ]; then
            for pattern in "${PATTERNS_CI[@]}"; do
                if printf '%s' "$line" | grep -Eiq "$pattern"; then
                    echo "❌ Potential secret detected in $file matching pattern: $pattern"
                    FOUND_SECRET=1
                    break
                fi
            done
        fi
    done < <(git diff --cached --unified=0 -- "$file" | grep '^+' | grep -v '^+++' | sed 's/^+//')
done <<< "$STAGED_FILES"

if [ "$FOUND_SECRET" -eq 1 ]; then
    echo ""
    echo "🚫 Commit blocked. Remove secrets before committing, or use 'git commit --no-verify' to bypass."
    exit 1
fi

echo "✅ No secrets detected. Proceeding with commit."
exit 0
 
 
Patterns Covered
Case‑sensitive patterns
Pattern
	
Description
AKIA[0-9A-Z]{16}	AWS Access Key ID
sk_live_[0-9a-zA-Z]{24}	Stripe live secret key
ghp_[0-9a-zA-Z]{36}	GitHub personal access token (classic)
gho_[0-9a-zA-Z]{36}	GitHub OAuth token
ghu_[0-9a-zA-Z]{36}	GitHub user-to-server token
ghs_[0-9a-zA-Z]{36}	GitHub server-to-server token
glpat-[0-9a-zA-Z]{20}	GitLab personal access token
xox[baprs]-[0-9a-zA-Z]{10,}	Slack token
AIza[0-9A-Za-z\-_]{35}	Google API key
-----BEGIN [A-Z]+ PRIVATE KEY-----	Private key block header
  
Case‑insensitive patterns
Pattern
	
Description
`(password	passwd
`(api[_-]?key	apikey
  
Customization

Edit the PATTERNS or PATTERNS_CI arrays inside the pre-commit file to add, remove, or modify detection rules.

     Use PATTERNS for case‑sensitive tokens (exact casing matters).
     Use PATTERNS_CI for case‑insensitive patterns like assignments where the variable name might be uppercase, lowercase, or mixed case.

Example:
bash
 
  
 
 
PATTERNS=(
  'AKIA[0-9A-Z]{16}'
  'my_custom_token_[0-9a-f]{32}'
)
 
 
Bypassing the Hook

In an emergency (e.g., a confirmed false positive), you can skip the hook using:
bash
 
  
 
 
git commit --no-verify
 
 

Use this sparingly and only when you are absolutely sure the match is not a real secret.
Limitations

     Scans staged changes only, not the entire file or repository history.
     Does not scan binary files – only text­-based diffs.
     Pattern matching is regex-­based and may occasionally produce false positives.
     Not a replacement for server-side scanning (e.g., GitHub Secret Scanning) – always revoke exposed credentials immediately if they leak.

License

This project is licensed under the MIT License. See the LICENSE file for details.

Copyright (c) [2026] [Job Done Marketing LLC]
