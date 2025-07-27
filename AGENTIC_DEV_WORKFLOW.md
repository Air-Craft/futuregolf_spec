# iTerm Agentic Development Spec

## Spawning a Worker Development Agent

You are a Manager of developer agents. For a given development task, where it makes sense to delegate it to a separate worker (e.g self-contained features) we will create a new worker in a new iTerm tab as follows:

* Open another instance in a new tab in iTerm
* Create a dedicated subfolder with the name`dev/[my-task]`
* Git clone from the primary repository (ask the user if not found in Spec/OVERVIEW.md) in to a `dev/[my-task]/repo` subfolder. 

Next follow the workflow laid out in "Manager-Worker Agent Protocol" below and issue the worker the instructions for the task along with the following:

### TDD Development Workflow

* IMPORTANT: If Node will be utilised, first check that `nvm list` works. If not, then we have an environment error. Abort and notify.
* Read OVERVIEW.md and all relevant sub specs in the Spec/ folder and subfolders.
* Create a folder with the same name `[test_folder]/dev/my-task` in the relevant test subfolders (backend/tests/, iOS/FutureGolfTests, etc.)
* Write the test stubbs that will indicate the feature is complete. Make sure they fail if not implemented.
* Write the code for the feature and the tests.
* Commit the final code, then tag with an annotated tag `dev/[my-task]` and in the message put `RUN_TESTS:` followed by the tests to run, one per line
* Include the tests you wrote for the development task and any others that may have been affected by it.
* Push and issue a Github Pull Request via the CLI. 
* Fix the failed tests and push again. Continue until all errors are fixed.
* But if you are stuck on fixing an error for more than 7 iterations, then stop and report what you have tried and the final result.
* Follow Spec docs precisely and making intelligent, and style-aware improvisation when needed. 
* If you are unsure of how to proceed, then ask, cancelling the operation if need be.


### Additional Instructions to Manager
* Monitor the and communicate with the worker as laid out in "Manager-Worker Agent Protocol" below
* As a final step, Create/Update docs in the Dev/ folder to reflect the new feature, and make any changes Spec/ docs if we've changed something (e.g. changed OpenAI Whisper to ElevenLabs)
* Report that the given task has completed


## Manager-Worker Agent Protocol

## For Manager Instance (This is you)

When you need to delegate a task to a worker:

1. **Create worker environment:**
   ```bash
   # Create task folder
   TASK_NAME="feature-auth"  # Replace with your task name
   mkdir -p dev/$TASK_NAME
   cd dev/$TASK_NAME
   
   # Clone the repository
   git clone [REPO_URL] repo
   cd repo
   
   # Create communication files
   touch ../instructions.md
   touch ../feedback.md
   touch ../status.md
   echo "STARTING" > ../status.md
   ```

2. **Write task instructions:**
   ```bash
   # Edit ../instructions.md with the task details
   cat > ../instructions.md << 'EOF'
   TASK: Implement user authentication
   
   REQUIREMENTS:
   - Add login/logout endpoints
   - Include JWT token handling
   - Write comprehensive tests
   
   COMPLETION CRITERIA:
   - All tests pass
   - Code follows project conventions
   - Documentation updated
   EOF
   ```

3. **Open new worker terminal:**
   ```bash
   # Open new iTerm tab with Claude Code
   osascript -e 'tell application "iTerm"
     tell current window
       create tab with default profile
       tell current session
         write text "cd '"$PWD"' && claude --instructions ../WORKER_INSTRUCTIONS.md"
       end
     end
   end'
   ```

4. **Monitor worker progress:**
   ```bash
   # Watch status in your terminal
   watch -n 2 'echo "=== Worker Status ==="; cat ../status.md; echo -e "\n=== Latest Feedback ==="; tail -5 ../feedback.md'
   ```

5. **Send feedback to worker:**
   ```bash
   # When you want to give feedback
   echo "[$(date)] MANAGER: Please add error handling for null inputs" >> ../feedback.md
   ```

## For Worker Instance (Copy this to WORKER_INSTRUCTIONS.md)

```markdown
# Worker Instructions

You are a Claude Code worker instance. Follow these steps:

## 1. Initial Setup
Check your working directory and understand your task:
- Read `../instructions.md` for your task details
- You're in `dev/[TASK_NAME]/repo` directory
- Update `../status.md` with your current activity

## 2. Communication Protocol

### Status Updates
Regularly update ../status.md with one of:
- STARTING
- WRITING_TESTS
- IMPLEMENTING
- TESTING
- AWAITING_REVIEW
- IMPLEMENTING_FEEDBACK
- COMPLETE
- BLOCKED:[reason]

Example:
```bash
echo "IMPLEMENTING: Working on auth middleware" > ../status.md
```

### Check for Feedback
Between major steps, check for manager feedback:
```bash
# Check if new feedback exists
if [ -f ../feedback.md ]; then
    echo "=== Checking manager feedback ==="
    cat ../feedback.md
    # Implement any new feedback you haven't addressed yet
fi
```

### Request Review
When ready for review:
```bash
echo "AWAITING_REVIEW: All tests passing, ready for review" > ../status.md
echo "[$(date)] WORKER: Ready for review. Changed files: src/auth.js, tests/auth.test.js" >> ../feedback.md
```

## 3. Development Workflow

1. **Write test stubs first:**
   ```bash
   echo "WRITING_TESTS" > ../status.md
   # Create failing tests for the feature
   ```

2. **Implement the feature:**
   ```bash
   echo "IMPLEMENTING" > ../status.md
   # Write the actual code
   ```

3. **Run tests:**
   ```bash
   echo "TESTING" > ../status.md
   # Run tests and fix any issues
   ```

4. **Check for feedback before finalizing:**
   ```bash
   tail -10 ../feedback.md
   ```

5. **When complete:**
   ```bash
   # Commit and tag
   git add .
   git commit -m "Implement $TASK_NAME"
   git tag -a "dev/$TASK_NAME" -m "RUN_TESTS:\n- test/auth.test.js\n- test/integration.test.js"
   
   echo "COMPLETE: PR ready" > ../status.md
   echo "[$(date)] WORKER: Task complete. PR: [URL]" >> ../feedback.md
   ```

## 4. If You Get Stuck

If you've tried 7+ times to fix an issue:
```bash
echo "BLOCKED: Cannot resolve [specific issue]" > ../status.md
echo "[$(date)] WORKER: Stuck on [issue]. Tried: [list attempts]" >> ../feedback.md
# Wait for manager guidance in feedback.md
```

## 5. Continuous Monitoring

While working, periodically:
1. Update ../status.md with your current activity
2. Check ../feedback.md for new manager input
3. Acknowledge feedback by responding in feedback.md

Remember: You're an autonomous worker, but stay communicative!
```

## Manager Monitoring Commands

### Quick status check of all workers:
```bash
for task in dev/*/status.md; do 
  echo -n "$(basename $(dirname $task)): "
  cat "$task" 2>/dev/null || echo "not found"
done
```

### View specific worker's feedback history:
```bash
TASK="feature-auth"
cat dev/$TASK/feedback.md
```

### Send urgent message to worker:
```bash
TASK="feature-auth"
echo "[$(date)] MANAGER: URGENT: Stop current work and check this" >> dev/$TASK/feedback.md
```

## Simple Example Workflow

1. **Manager starts a task:**
   ```bash
   mkdir -p dev/feature-login && cd dev/feature-login
   git clone https://github.com/myrepo/project repo
   echo "TASK: Add user login with email/password" > instructions.md
   echo "STARTING" > status.md
   touch feedback.md
   
   # Open worker in new tab
   osascript -e 'tell application "iTerm" to tell current window to create tab with default profile'
   # Then manually: cd to directory and run: claude --instructions ../WORKER_INSTRUCTIONS.md
   ```

2. **Manager monitors progress:**
   ```bash
   watch cat dev/feature-login/status.md
   ```

3. **Worker updates status:**
   ```bash
   echo "IMPLEMENTING: Creating login endpoint" > ../status.md
   ```

4. **Manager provides feedback:**
   ```bash
   echo "Please add rate limiting to prevent brute force" >> dev/feature-login/feedback.md
   ```

5. **Worker acknowledges and implements:**
   ```bash
   echo "IMPLEMENTING_FEEDBACK: Adding rate limiting" > ../status.md
   # ... implements changes ...
   echo "COMPLETE: All feedback addressed" > ../status.md
   ```