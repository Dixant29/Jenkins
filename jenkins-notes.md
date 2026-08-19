# Jenkins Notes

## What a Jenkins Pipeline does

A Jenkins Pipeline automates the path from source code to a working release:

```text
Checkout -> Build -> Test -> Deploy
```

The pipeline is normally stored in a `Jenkinsfile` at the root of the repository. It tells Jenkins **which commands to run and in which order**; it does not contain the application itself.

## What to start with

Start with one tiny pipeline and make it run successfully. Create a file named `Jenkinsfile` containing:

```groovy
pipeline {
  agent any

  stages {
    stage('Hello') {
      steps {
        echo 'Hello from Jenkins'
      }
    }
  }
}
```

Then create a Jenkins **Pipeline** job, configure it to use this repository's `Jenkinsfile`, and click **Build Now**. Open the Console Output and confirm that it prints `Hello from Jenkins`.

Once this works, add stages one at a time:

1. `Checkout` — retrieve the repository code.
2. `Build` — run your project's build command.
3. `Test` — run its test command.
4. `Deploy` — add this only after build and test are reliable.

Do not add the next stage until the current pipeline runs successfully.

## The syntax and languages

`Jenkinsfile` uses Jenkins Pipeline syntax, which is based on Groovy. Prefer the structured **Declarative Pipeline** style above when starting out.

The commands Jenkins executes can use any tools installed on its agent. For example:

```groovy
sh 'npm test'          // Node.js
sh 'python -m pytest'  // Python
sh './mvnw test'       // Java / Maven
sh './gradlew test'    // Java / Kotlin / Gradle
bat 'dotnet test'      // Windows / .NET
```

## Compact learning plan

1. Learn the flow: checkout -> build -> test -> deploy.
2. Run the one-stage `Hello` pipeline above.
3. Replace `echo` with your project's real build command.
4. Add its test command in a separate `Test` stage.
5. Learn only the next feature you need: Git, environment variables, credentials, artifacts, or deployment.

## Remembering it

- Keep this file beside the working `Jenkinsfile`.
- After each change, read the Jenkins Console Output and write down the error and its fix.
- Recreate the small pipeline from memory after a few days.
- Change one thing at a time and run it.

## Safety reminder

Never place passwords, API tokens, or private keys directly in a `Jenkinsfile`. Store them in Jenkins Credentials and reference them from the pipeline.



1. need to connect jenkins to github
2.