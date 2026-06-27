# 25 Cheat Sheets

This document compiles cheat sheets for daily engineering work: Git versioning, Docker container commands, SQL database queries, Linux terminal commands, React hooks lifecycle, and HTTP status codes.

## Git Commands Cheat Sheet

### Definition
Git commands manage code commits, tracking, branching, and synchronization. The cheat sheet compiles syntax commands for daily codebase operations.

### Real-world Analogy
Think of a physical filing office. A git checkout is changing your desk nameplate. A git branch is creating a duplicate ledger. A git commit is stamping the ledger page, and a git push is mailing the updated pages to the regional branch office.

### Code Example
```bash
# Branch and Commit Commands
git checkout -b feat/new-page     # Create and switch to branch
git add .                         # Stage all changes
git commit -m "feat: login page"  # Commit locally
git push -u origin feat/new-page  # Push and track remote
```

### Common Interview Questions
- What is the difference between git pull and git fetch?
- How do you undo the last commit but keep the local changes (e.g. git reset --soft HEAD~1)?
- How do you rename a branch locally and on the remote?

### Reference Links
- [Git: Reference Guide](https://git-scm.com/docs)
- [GitHub: Git Cheat Sheet PDF](https://training.github.com/downloads/github-git-cheat-sheet.pdf)

## Docker Commands Cheat Sheet

### Definition
Docker commands manage image compilation, container execution, networks, volumes, and status inspections. The cheat sheet compiles syntax commands for daily container administration.

### Real-world Analogy
Imagine running a shipping warehouse. Building an image is packing a crate template. Running a container is releasing the automated delivery vehicle onto the floor. Volume management is setting up persistent loading docks, and network management is assigning communication radios to drivers.

### Code Example
```bash
# Docker Engine Operations
docker build -t myapp:latest .       # Build image
docker run -d -p 8080:80 myapp       # Run container in background
docker ps                            # List running containers
docker system prune -a --volumes     # Clear unused space
```

### Common Interview Questions
- Compare docker stop and docker kill.
- How do you view container console logs in real-time (e.g. docker logs -f)?
- How do you run commands inside an active container (e.g. docker exec)?

### Reference Links
- [Docker Docs: CLI Reference](https://docs.docker.com/engine/reference/commandline/cli/)
- [Docker: CLI Cheat Sheet](https://docs.docker.com/get-started/docker_cheatsheet.pdf)

## SQL Database Commands Cheat Sheet

### Definition
SQL commands manipulate relational databases. The cheat sheet compiles commands for table design, row querying, joins, index creation, and transaction management.

### Real-world Analogy
Imagine managing a real-estate database. Table design is establishing the layout of houses. Selecting rows is searching for listings under a certain price. Joins are matching tenants with their apartment keys.

### Code Example
```sql
-- SQL Query Reference
SELECT users.id, profiles.bio
FROM users
INNER JOIN profiles ON users.id = profiles.user_id
WHERE users.active = true
ORDER BY users.created_at DESC;
```

### Common Interview Questions
- What is the difference between INNER JOIN, LEFT JOIN, and RIGHT JOIN?
- How do you update rows and check constraints?
- What does the CASCADE constraint enforce in tables?

### Reference Links
- [PostgreSQL Docs: SQL Commands](https://www.postgresql.org/docs/current/sql-commands.html)
- [W3Schools: SQL Reference](https://www.w3schools.com/sql/)

## Linux Terminal Commands Cheat Sheet

### Definition
Linux terminal commands navigate file paths, check system statuses, configure permissions, and manage active processes. The cheat sheet compiles commands for command-line server operations.

### Real-world Analogy
Using the terminal is like exploring a house in total darkness using a flashlight. Instead of opening doors with a mouse click (GUI), you announce where you want to go using voice commands (CLI) to walk through directories and read documents.

### Code Example
```bash
# Linux Path and Process Commands
pwd                          # Show current directory path
ls -la                       # List all files and permissions
chmod 400 key.pem            # Set read-only file permissions
ps aux | grep node           # Find running node processes
kill -9 1042                 # Force terminate process ID 1042
```

### Common Interview Questions
- How do you monitor memory and CPU usage in real-time (e.g. top or htop)?
- Explain what file permissions like chmod 755 represent.
- Compare systemctl and service commands.

### Reference Links
- [Ubuntu: Linux Command Line Basics](https://ubuntu.com/tutorials/command-line-for-beginners)
- [Linux Command Manuals](https://linuxmanpages.com/)

## React Hooks Lifecycle Cheat Sheet

### Definition
React Hooks manage component states and side-effects. The cheat sheet outlines hooks execution order, dependency triggers, and cleanup executions.

### Real-world Analogy
Think of a plant lifecycle. useState is setting the seed pot. useEffect is setting automatic water spraying at 9 AM (effect). The dependency array is checking: "Spray water ONLY if the soil temperature changes". The cleanup function is sweeping up dry leaves before planting a new seed.

### Code Example
```jsx
// React Hooks lifecycle templates
import { useState, useEffect } from "react";

export function TimerComponent() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    // Mount phase: Start interval
    const interval = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);

    // Unmount/Cleanup phase: Clear interval before re-render
    return () => clearInterval(interval);
  }, []); // Empty array runs effect only once on mount

  return <div>Time: {seconds}</div>;
}
```

### Common Interview Questions
- What happens if you omit the dependency array in useEffect?
- Explain the execution flow of the cleanup function in useEffect.
- How does useLayoutEffect differ from useEffect?

### Reference Links
- [React Docs: Built-in Hooks](https://react.dev/reference/react)
- [React Docs: Synchronizing with Effects](https://react.dev/learn/synchronizing-with-effects)

## HTTP Status Codes Cheat Sheet

### Definition
HTTP status codes represent server responses to client requests. The cheat sheet compiles codes across standard response ranges (2xx, 3xx, 4xx, 5xx).

### Real-world Analogy
HTTP status codes are hand signals used by a traffic director. Thumbs up (200 OK) means cross the street. Redirecting traffic (301 Redirect) means take the side road. Shaking their head (404 Not Found) means the street does not exist. A shrug (500 Server Error) means the bridge collapsed.

### Code Example
```
// HTTP Response Ranges Summary:
// 1xx (Informational): Request received, processing.
// 2xx (Success): Request processed successfully (e.g., 200 OK, 201 Created).
// 3xx (Redirection): Further action needed (e.g., 301 Permanent, 302 Found).
// 4xx (Client Error): Request has bad syntax (e.g., 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found).
// 5xx (Server Error): Server failed to fulfill request (e.g., 500 Internal Error, 502 Bad Gateway, 503 Service Unavailable).
```

### Common Interview Questions
- What is the difference between 401 Unauthorized and 403 Forbidden?
- In what scenarios is a 502 Bad Gateway returned instead of a 500 Internal Server Error?
- What does the 429 Too Many Requests status code enforce?

### Reference Links
- [MDN Web Docs: HTTP Status Codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
- [RFC 7231: HTTP/1.1 Semantics and Content](https://datatracker.ietf.org/doc/html/rfc7231)

## Scenario-based Interview Questions for Cheat Sheets

### Scenario 1
You run git push and get a "Rejected: non-fast-forward" error message. The remote repository has new commits pushed by your teammates. You do not want to wipe out your local commits. How do you resolve this?

*Expected Approach:*
1. Explain that the error happens because your local branch is behind the remote tracking branch.
2. Run git fetch to update your local copy of the remote history.
3. Run git rebase origin/main (or git pull --rebase) to apply your local commits on top of the remote commits.
4. Resolve any merge conflicts, and run git push to push the clean, linear history.

### Scenario 2
Your client-side application requests data from your API server. The browser network panel shows the request status as "Cancelled" or returning status 400 with a console error: "Origin http://localhost:3000 is not allowed by Access-Control-Allow-Origin". How do you resolve this?

*Expected Approach:*
1. Identify this as a Cross-Origin Resource Sharing (CORS) security block.
2. Fix this on the backend Express application by installing the cors middleware package.
3. Configure the cors middleware options to whitelist http://localhost:3000 as an allowed origin, returning the correct Access-Control-Allow-Origin header to the browser.
