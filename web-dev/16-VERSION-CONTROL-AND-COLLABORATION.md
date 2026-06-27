# 16 Version Control and Collaboration

This guide covers version control, Git internals, branching strategies, merge vs rebase, git hooks (Husky), pull request practices, and archaeology commands.

## Git Fundamentals

### Definition
Git is a distributed version control system that tracks file changes. The architecture is composed of four stages: the Working Directory (untracked/modified files), the Staging Area or Index (files marked for commit), the Local Repository (committed commits), and the Remote Repository (cloud-hosted database).

### Real-world Analogy
Imagine taking photos for a yearbook. The Working Directory is editing photos on your computer desktop. The Staging Area is placing 5 photos in a specific layout folder. The Local Repository is printing the yearbook and saving the copy on your office shelf. The Remote Repository is uploading the yearbook PDF to the publisher's cloud account.

### Code Example
```bash
# Add file to Staging Area, commit locally, and push to remote
git add index.js
git commit -m "feat: implement database connection"
git push origin main
```

### Common Interview Questions
- Describe the transitions between the four stages of Git.
- What is the difference between git checkout and git switch?
- How does Git store objects under the hood (e.g. blobs, trees, commits)?

### Reference Links
- [Git: Pro Git Book](https://git-scm.com/book/en/v2)
- [Git: Git Objects](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects)

## Git Workflow Models

### Definition
Git workflows govern team collaboration. Models include Git Flow (strict develop, release, hotfix, and master branches), GitHub Flow (simple main branch with short-lived feature branches and Pull Requests), and Trunk-Based Development (developers merge small commits to main frequently).

### Real-world Analogy
Git Flow is like building a car: departments work in silos, compile a prototype model (develop), test it on a track (release), and launch it to the public showroom (master). Trunk-Based is like a community sandbox: everyone builds the sandcastle together, adding small bucket loads directly to the main castle every 15 minutes.

### Code Example
```bash
# Typical GitHub Flow sequence
git checkout -b feat/user-login
# ...make changes...
git add .
git commit -m "feat: user login interface"
git push origin feat/user-login
# ...open Pull Request on GitHub...
```

### Common Interview Questions
- Contrast GitHub Flow and Git Flow. When is each appropriate?
- What are the main benefits and challenges of Trunk-Based Development?
- How do hotfix branches operate in Git Flow?

### Reference Links
- [GitHub Flow Guide](https://docs.github.com/en/get-started/using-git/about-git-workflows/github-flow)
- [Trunk Based Development Website](https://trunkbaseddevelopment.com/)

## Merge vs Rebase

### Definition
Merge combines branches, creating a new merge commit and preserving the historical chronology. Rebase rewrites commits on top of the target branch, creating a linear history. Interactive Rebase (squash, fixup) cleans up local commit records.

### Real-world Analogy
Merge is like joining two streams: they flow together, and you see the exact point where they met (merge commit). Rebase is like copying the blocks of your house, building a new base foundation, and stacking your blocks on top of the new foundation, making a single tall tower.

### Code Example
```bash
# Rebasing a feature branch onto main
git checkout feat/auth
git rebase main

# Safe force-push after rebase
git push origin feat/auth --force-with-lease
```

### Common Interview Questions
- Why is rebasing public shared branches considered an anti-pattern?
- Compare git merge and git rebase in terms of git log history.
- What does git push --force-with-lease enforce, and why is it safer than --force?

### Reference Links
- [Git: Rebasing](https://git-scm.com/book/en/v2/Git-Branching-Rebasing)
- [Atlassian: Merge vs Rebase](https://www.atlassian.com/git/tutorials/merging-vs-rebasing)

## Git Hooks

### Definition
Git Hooks are scripts that run automatically before or after Git events (e.g. pre-commit, pre-push). Tools like Husky and lint-staged configure hooks in project files to automate linting, testing, and formatting before commits are finalized.

### Real-world Analogy
Imagine submitting an application form at a bank. Before you hand the paper to the teller, a machine scans the form to check if you signed the signature line (pre-commit check). If it is missing, the machine hands the form back and blocks you from standing in the teller queue.

### Code Example
```json
// package.json Husky config example
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "*.js": [
      "eslint --fix",
      "prettier --write"
    ]
  }
}
```

### Common Interview Questions
- What are Git Hooks and where are they stored locally in a project?
- How do Husky and lint-staged work together to enforce code styling rules?
- How do you bypass Git Hooks when committing (e.g. --no-verify)?

### Reference Links
- [Husky Documentation](https://typicode.github.io/husky/)
- [Git: Customizing Git Hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)

## Pull Request Best Practices

### Definition
Pull Requests (PRs) facilitate code review before merging changes into the target branch. Best practices include keeping PR sizes small, using descriptive templates, conducting reviews, resolving comments, and using squash-and-merge.

### Real-world Analogy
Imagine editing a chapter in a book. Instead of sneaking into the printing press and swapping pages in the master copy, you print your edits on a draft page, stick it on the editor's door, and wait for them to review, highlight changes, and sign off.

### Code Example
```markdown
# Sample Pull Request Template (PR_TEMPLATE.md)
## Description
- Implement JWT session verification on middleware.

## Checklist
- [x] Unit tests passing
- [x] Code conforms to styling guides
- [x] Linked issue: #102
```

### Common Interview Questions
- What are the benefits of using Squash and Merge when merging Pull Requests?
- What are draft PRs and how do they benefit team communication?
- Describe the characteristics of an effective code review comment.

### Reference Links
- [GitHub: About Pull Requests](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests)

## Git Archaeology

### Definition
Git Archaeology refers to using advanced Git command operations (git log, git blame, git bisect, git reflog, git cherry-pick, git stash) to investigate history, search commit states, trace bugs, and recover lost commits.

### Real-world Analogy
git reflog is like a security camera recording the librarian's movements. Even if they accidentally shred a folder (hard delete a branch), the camera tape shows: "Librarian dropped folder in shredder at 2 PM", letting you trace the barcode number and reconstruct the folder (recover commit).

### Code Example
```bash
# Recover a lost commit after an accidental hard reset
git reflog
# Find the commit hash from reflog list (e.g., e3a4f10)
git reset --hard e3a4f10

# Binary search to locate which commit introduced a bug
git bisect start
git bisect bad         # Current commit is broken
git bisect good v1.0.0 # Release 1.0.0 was working
```

### Common Interview Questions
- What is git reflog and how does it save you from accidental commit data loss?
- Describe how git bisect uses binary search algorithms to locate bugs.
- Compare git cherry-pick and git merge.

### Reference Links
- [Git: Reflog](https://git-scm.com/docs/git-reflog)
- [Git: Bisect](https://git-scm.com/docs/git-bisect)

## Scenario-based Interview Questions for Version Control

### Scenario 1
You accidentally ran git reset --hard HEAD~1, deleting your last three commits containing a full day's work. The changes are not pushed to the remote repository. How do you recover your work?

*Expected Approach:*
1. Explain that Git does not delete commits immediately; it removes pointers, and the commits remain in database loose objects until garbage collected.
2. Run git reflog to display a log of all recent HEAD changes.
3. Locate the commit hash before the hard reset was run.
4. Run git reset --hard <commit-hash> to restore the branch pointer, recovering the work.

### Scenario 2
Your team's main branch is constantly broken. Developers merge long-lived feature branches (containing 50+ commits) that fail automated tests and trigger dozens of merge conflicts. How do you resolve this?

*Expected Approach:*
1. Propose transition to Trunk-Based Development or enforce smaller, short-lived feature branches (1-2 days max).
2. Integrate a CI build pipeline that runs automated linting and tests on every Pull Request, blocking merges if tests fail.
3. Require developers to run interactive rebases (git rebase -i) to clean up local history, squashing loose commits into clean, logical changes before merging.
