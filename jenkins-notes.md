# Jenkins Learning Notes

## Current state — resume here

The learning Jenkins Pipeline is working successfully from GitHub.

- Local Jenkins is accessed at `http://localhost:8080/`.
- GitHub repository: `https://github.com/Dixant29/Jenkins/`
- Branch: `main` (not `master`).
- Jenkins job: `learning`.
- Job configuration: **Pipeline script from SCM** -> **Git** -> repository above -> branch `*/main` -> Script Path `Jenkinsfile`.
- A successful build retrieved `Jenkinsfile` from GitHub, checked out `main`, printed `Hello to jenkins`, and finished with `SUCCESS`.
- The current `Jenkinsfile` has `Hello`, `Build`, and `Test` stages plus a `post { always { ... } }` block. Build and Test are still placeholder `echo` commands.
- **Next task:** replace the Build and Test placeholders with the shell exercise in [Next pipeline change](#next-pipeline-change), commit, push, and run the Jenkins job.

## What Jenkins Pipeline is

A Jenkins Pipeline automates a sequence such as:

```text
Checkout -> Build -> Test -> Deploy
```

It is normally defined in a root-level `Jenkinsfile`:

```text
repository-root/
├── Jenkinsfile
├── jenkins-notes.md
├── src/
└── tests/
```

The Jenkinsfile does **not** contain the application code. It orchestrates the commands and tools that build, test, and deploy the application.

## Jenkinsfile syntax and languages

`Jenkinsfile` uses Jenkins Pipeline syntax based on Groovy. Start with the structured **Declarative Pipeline** style:

```groovy
pipeline {
  agent any

  stages {
    stage('Hello') {
      steps {
        echo 'Hello to Jenkins'
      }
    }
  }
}
```

Jenkins can execute any tools installed on its agent:

```groovy
sh 'npm test'          // Node.js
sh 'python -m pytest'  // Python
sh './mvnw test'       // Java / Maven
sh './gradlew test'    // Java / Kotlin / Gradle
bat 'dotnet test'      // Windows / .NET
```

## Local Jenkins setup

The official Jenkins website provides downloads and documentation; it does not run a personal pipeline directly. Jenkins must run on a local machine, server, or hosted service.

For this local setup, Jenkins runs at:

```text
http://localhost:8080/
```

During initial setup, the administrator password can be read on Linux with:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Do not share this password. If the file cannot be found, check logs:

```bash
sudo journalctl -u jenkins --no-pager -n 100
```

VS Code edits files. Jenkins is controlled through its browser dashboard (or later through API/CLI/webhooks).

## Git, GitHub, and SCM

**SCM** means **Source Control Management**: software that tracks changes to source files over time.

- **Git** is the source-control tool.
- **GitHub** hosts Git repositories online.
- Jenkins using **Pipeline script from SCM** means it obtains the pipeline definition and project source from the Git repository.

Normal workflow:

```text
Edit Jenkinsfile in VS Code
-> git add Jenkinsfile
-> git commit -m "Describe the change"
-> git push
-> Jenkins: Build Now
-> inspect Console Output
```

`git add Jenkinsfile` puts the current Jenkinsfile change in Git's staging area so the next commit includes it. `git commit` creates a local history checkpoint; `git push` uploads commits to GitHub.

Useful commands:

```bash
git status             # show changed/staged files and branch state
git pull origin main   # download and merge changes made on GitHub
git add Jenkinsfile
git commit -m "Describe the change"
git push
```

Edit locally in VS Code. Use `git pull origin main` before work only if GitHub (or another person) may have changed the repository.

## Jenkins configuration and the first error fixed

Create a Jenkins **Pipeline** job, then configure:

```text
Definition: Pipeline script from SCM
SCM: Git
Repository URL: https://github.com/Dixant29/Jenkins/
Branches to build: */main
Script Path: Jenkinsfile
```

The first SCM error was:

```text
fatal: couldn't find remote ref refs/heads/master
```

Cause: Jenkins was set to `*/master`, but GitHub uses `main`. Changing it to `*/main` fixed the problem.

## Why Jenkins gets Jenkinsfile first, then checks out the repository

Jenkins retrieves the Jenkinsfile first so it can learn what pipeline exists and how/where to run it. Then Declarative Pipeline performs an automatic checkout into the build workspace so Build/Test commands can use the source code.

The successful log showed a workspace similar to:

```text
/var/lib/jenkins/workspace/learning
```

The automatic stage is named `Declarative: Checkout SCM`. On later builds Jenkins commonly reuses the workspace and fetches changes instead of cloning from scratch.

The standard checkout normally creates a working copy of the repository. This is the right default for small repositories. Large monorepos can later use sparse checkout to obtain only needed paths.

To control checkout yourself later:

```groovy
options {
  skipDefaultCheckout()
}
```

Then add an explicit `checkout scm` step. Do not use this yet without a specific need.

One repository can have multiple pipeline files. Create one Jenkins job per pipeline and set each job's Script Path, for example:

```text
Jenkinsfile
pipelines/Jenkinsfile.backend
pipelines/Jenkinsfile.frontend
pipelines/Jenkinsfile.release
```

Separate repositories are not required merely because there are multiple pipelines.

## Declarative Pipeline concepts learned

```groovy
pipeline {                 // the complete Declarative Pipeline
  agent any                // run on any available Jenkins agent

  stages {                 // collection of ordered stages
    stage('Build') {       // named phase shown in Jenkins UI
      steps {              // commands/actions in that phase
        echo 'message'     // print a message in Console Output
      }
    }
  }

  post {                   // optional actions after pipeline completion
    always {               // runs whether successful, failed, or aborted
      echo 'Pipeline has finished'
    }
  }
}
```

`post` is optional. It cannot contain an `echo` directly; it needs a condition block:

```groovy
post {
  success { echo 'Done' }
  failure { echo 'Pipeline failed' }
  always  { echo 'Cleanup or report here' }
}
```

Typical later uses: test-result publishing, cleanup, and notifications.

## Next pipeline change

Learn `sh`, which runs Linux shell commands on the Jenkins agent. Replace the current placeholder contents of the Build and Test stages with:

```groovy
stage('Build') {
  steps {
    sh '''
      echo "Building from: $(pwd)"
      ls -la
      mkdir -p output
      echo "Build completed" > output/build-info.txt
    '''
  }
}

stage('Test') {
  steps {
    sh '''
      test -f output/build-info.txt
      echo "Test passed"
    '''
  }
}
```

Expected result: the log shows the workspace, a file listing, and `Test passed`.

### Shell-command glossary

- `sh ''' ... '''`: execute enclosed Linux shell commands.
- `pwd`: print the current working directory.
- `$(pwd)`: run `pwd` and substitute its output into surrounding text.
- `ls -la`: list files. `-l` means detailed/long format; `-a` includes hidden files such as `.git`.
- `mkdir -p output`: create `output`; `-p` creates missing parent folders and does not fail if the directory already exists.
- `>`: redirect text into a file, creating or **overwriting** it.
- `echo "Build completed" > output/build-info.txt`: writes `Build completed` to that file.
- `test -f output/build-info.txt`: succeeds only if the path exists and is a regular file.
- A **regular file** is a normal data file (such as Jenkinsfile, README.md, or build-info.txt), not a directory or special operating-system object.
- `test -d output`: check whether `output` is a directory.
- `test -e path`: check whether a path exists.

If `test -f` fails, Jenkins stops the shell step and marks the Test stage/pipeline failed. It prints nothing on success, so `echo "Test passed"` confirms it succeeded.

## Learning sequence

Completed:

1. Installed and unlocked local Jenkins.
2. Created and ran a Hello Pipeline.
3. Put the Jenkinsfile under Git/GitHub source control.
4. Configured Jenkins to run the pipeline from SCM on `main`.
5. Added Build, Test, and `post { always { ... } }` structure.

Next:

1. Run the `sh` Build/Test exercise above.
2. Archive `output/build-info.txt` as a Jenkins artifact.
3. Replace the learning commands with real build/test commands for an actual project.
4. Learn environment variables and Jenkins Credentials.
5. Add deployment only after build and test are reliable.

## Learning habits and safety

- Make one small change at a time, run it, and read Console Output.
- Keep this note beside the working Jenkinsfile.
- Record errors and fixes; they are useful learning material.
- Recreate small examples from memory after a few days.
- Never put passwords, API tokens, or private keys in a Jenkinsfile. Store them in Jenkins Credentials and reference them securely.
