# Jenkins Shared Library for Reusable Pipeline Stages

A reusable Jenkins Shared Library written in Groovy for standardizing common CI/CD pipeline steps and reducing duplicated logic across Jenkins pipelines.

This project demonstrates how Jenkins Shared Libraries can be used to define reusable pipeline functions under the `vars/` directory and call them from different Jenkinsfiles.

---

## Table of Contents

- [Overview](#overview)
- [Project Purpose](#project-purpose)
- [What Is a Jenkins Shared Library?](#what-is-a-jenkins-shared-library)
- [Repository Structure](#repository-structure)
- [Available Shared Library Functions](#available-shared-library-functions)
- [How to Configure the Library in Jenkins](#how-to-configure-the-library-in-jenkins)
- [How to Use the Library in a Jenkinsfile](#how-to-use-the-library-in-a-jenkinsfile)
- [Example Jenkinsfile](#example-jenkinsfile)
- [Current Implementation Notes](#current-implementation-notes)
- [Recommended Improvements](#recommended-improvements)
- [Troubleshooting](#troubleshooting)
- [Skills Demonstrated](#skills-demonstrated)
- [Author](#author)
- [License](#license)

---

## Overview

This repository contains a basic Jenkins Shared Library that can be imported into Jenkins pipelines.

Instead of repeating the same logic in every `Jenkinsfile`, common pipeline actions can be written once in this repository and reused across multiple projects.

For example, a pipeline can call a shared function such as:

```groovy
helloWorld('Ofri')
```

or:

```groovy
checkoutCode('https://github.com/example/repository.git', 'main')
```

This approach helps make Jenkins pipelines cleaner, easier to maintain, and more consistent.

---

## Project Purpose

The purpose of this project is to demonstrate:

- How Jenkins Shared Libraries are structured.
- How reusable pipeline functions are created in Groovy.
- How Jenkinsfiles can call shared functions.
- How repeated CI/CD logic can be centralized in one repository.
- How pipeline code can become cleaner and easier to maintain.

This is especially useful when managing multiple DevOps projects that share similar stages such as:

- Checkout source code
- Build application
- Run tests
- Build Docker images
- Push images to a registry
- Deploy to Kubernetes
- Send notifications

---

## What Is a Jenkins Shared Library?

A Jenkins Shared Library is a separate Git repository that contains reusable pipeline code.

Instead of writing all pipeline logic directly inside each `Jenkinsfile`, shared logic can be stored in a library and imported into pipelines.

A common Jenkins Shared Library structure includes:

```text
(root)
├── vars/
│   └── reusableStep.groovy
├── src/
│   └── package/classes
└── resources/
    └── supporting files
```

In this project, the library currently uses the `vars/` directory.

Files inside `vars/` become global pipeline steps that can be called directly from a Jenkinsfile.

Example:

```text
vars/helloWorld.groovy
```

can be used in a Jenkinsfile as:

```groovy
helloWorld('Ofri')
```

---

## Repository Structure

```text
jenkins-shared-library-pipeline-utils/
│
├── README.md
│
└── vars/
    ├── checkoutCode.groovy
    └── helloWorld.groovy
```

### Directory Explanation

| Path | Description |
|---|---|
| `vars/` | Contains reusable global pipeline functions |
| `vars/helloWorld.groovy` | Simple test function that prints a greeting |
| `vars/checkoutCode.groovy` | Reusable function for cloning a Git repository and checking out a branch |
| `README.md` | Project documentation |

---

## Available Shared Library Functions

## 1. `helloWorld`

### File

```text
vars/helloWorld.groovy
```

### Purpose

A simple test function used to verify that the Jenkins Shared Library is loaded correctly.

### Current Function Logic

```groovy
def call(name = 'World') {
    echo "Hello, ${name}!"
}
```

### Usage Example

```groovy
helloWorld('Ofri')
```

### Expected Jenkins Console Output

```text
Hello, Ofri!
```

This function is useful as a first test after configuring the shared library in Jenkins.

---

## 2. `checkoutCode`

### File

```text
vars/checkoutCode.groovy
```

### Purpose

A reusable function for cloning a Git repository and checking out a specific branch.

### Current Function Logic

```groovy
def call(String repoUrl, String branch) {
    def workingDir = "${env.WORKSPACE}"

    bat "git clone ${repoUrl} ${workingDir}"
    bat "git checkout ${branch}"

    return workingDir
}
```

### Parameters

| Parameter | Description | Example |
|---|---|---|
| `repoUrl` | Git repository URL to clone | `https://github.com/example/repo.git` |
| `branch` | Branch to check out after cloning | `main` |

### Usage Example

```groovy
checkoutCode('https://github.com/libermanofri/example-project.git', 'main')
```

### What It Does

```text
1. Uses the Jenkins workspace as the working directory.
2. Clones the requested Git repository.
3. Checks out the requested branch.
4. Returns the workspace path.
```

---

## How to Configure the Library in Jenkins

To use this shared library in Jenkins:

1. Open Jenkins.
2. Go to **Manage Jenkins**.
3. Open **System**.
4. Find **Global Trusted Pipeline Libraries**.
5. Add a new library.
6. Configure the library name.

Recommended library name:

```text
pipeline-utils
```

7. Set the default branch:

```text
main
```

8. Set the project repository URL:

```text
https://github.com/libermanofri/jenkins-shared-library-pipeline-utils.git
```

9. Save the configuration.

After this, Jenkins pipelines can import the library using:

```groovy
@Library('pipeline-utils') _
```

---

## How to Use the Library in a Jenkinsfile

At the top of the Jenkinsfile, import the library:

```groovy
@Library('pipeline-utils') _
```

Then call the shared functions inside pipeline stages.

Example:

```groovy
pipeline {
    agent any

    stages {
        stage('Test Shared Library') {
            steps {
                helloWorld('Ofri')
            }
        }
    }
}
```

---

## Example Jenkinsfile

```groovy
@Library('pipeline-utils') _

pipeline {
    agent any

    stages {
        stage('Print Greeting') {
            steps {
                helloWorld('Ofri')
            }
        }

        stage('Checkout Code') {
            steps {
                script {
                    checkoutCode('https://github.com/libermanofri/example-project.git', 'main')
                }
            }
        }
    }
}
```

---

## Current Implementation Notes

### Windows-Based Jenkins Agent

The current `checkoutCode` function uses:

```groovy
bat
```

This means the function is currently designed for a Jenkins agent running on Windows.

For a Linux-based Jenkins agent, the command should use:

```groovy
sh
```

instead of:

```groovy
bat
```

Example Linux version:

```groovy
def call(String repoUrl, String branch) {
    def workingDir = "${env.WORKSPACE}"

    sh "git clone ${repoUrl} ${workingDir}"
    sh "git checkout ${branch}"

    return workingDir
}
```

---

### Workspace Behavior

The function clones the repository directly into:

```groovy
env.WORKSPACE
```

This can be useful in simple pipelines, but in some cases Jenkins may already have files inside the workspace.

In a more advanced version, the function can be improved to clean the workspace before cloning or clone into a subdirectory.

---

### Git Checkout Location

The current function runs:

```groovy
bat "git checkout ${branch}"
```

after cloning.

Depending on the Jenkins workspace and shell behavior, it may be safer to run the checkout command from inside the cloned repository directory.

A future improvement can adjust the function to use:

```groovy
dir(workingDir) {
    bat "git checkout ${branch}"
}
```

---

## Recommended Improvements

This project is intentionally simple, but it can be expanded into a stronger Jenkins Shared Library.

### 1. Add Linux Agent Support

Currently, the checkout function uses Windows `bat` commands.

A useful improvement would be to support both Windows and Linux agents.

Example approach:

```groovy
if (isUnix()) {
    sh "git clone ${repoUrl} ${workingDir}"
} else {
    bat "git clone ${repoUrl} ${workingDir}"
}
```

Benefits:

- Works on both Windows and Linux Jenkins agents.
- Makes the library more reusable.
- Better fits real CI/CD environments.

---

### 2. Add Input Validation

The functions can be improved by checking whether required parameters were provided.

Example:

```groovy
if (!repoUrl?.trim()) {
    error "repoUrl is required"
}
```

Benefits:

- Clearer pipeline errors.
- Easier troubleshooting.
- Prevents confusing failures.

---

### 3. Add More Reusable Pipeline Steps

The library can be expanded with additional CI/CD helper functions.

Possible future functions:

```text
runTests.groovy
buildDockerImage.groovy
pushDockerImage.groovy
deployWithHelm.groovy
sendNotification.groovy
```

Benefits:

- Less duplicated Jenkinsfile code.
- More consistent CI/CD workflows.
- Easier pipeline maintenance.

---

### 4. Add Documentation for Each Function

Jenkins Shared Libraries can include matching `.txt` help files for global variables.

Example:

```text
vars/helloWorld.txt
vars/checkoutCode.txt
```

Benefits:

- Better Jenkins documentation.
- Easier usage for other developers.
- More professional shared library structure.

---

### 5. Add Unit Tests for Shared Library Code

The project can be improved by adding testing for Groovy pipeline code.

Possible tools:

- Jenkins Pipeline Unit
- Groovy unit tests

Benefits:

- Safer shared library changes.
- Reduced risk of breaking multiple pipelines.
- More production-like CI/CD library management.

---

## Troubleshooting

### 1. Jenkins Cannot Find the Shared Library

Check:

- The library is configured under **Global Trusted Pipeline Libraries**.
- The library name in Jenkins matches the import name in the Jenkinsfile.
- The branch name is correct.
- Jenkins has access to the GitHub repository.

Example import:

```groovy
@Library('pipeline-utils') _
```

---

### 2. Function Not Found

If Jenkins cannot find a function such as:

```groovy
helloWorld()
```

check that the file exists under:

```text
vars/helloWorld.groovy
```

The function name is based on the filename inside the `vars/` directory.

---

### 3. `bat` Command Fails on Linux Agent

The current checkout function uses:

```groovy
bat
```

This only works on Windows agents.

For Linux agents, replace `bat` with `sh`, or improve the function with `isUnix()`.

---

### 4. Git Clone Fails

Check:

- The repository URL is correct.
- Jenkins has network access to GitHub.
- Git is installed on the Jenkins agent.
- The target workspace is clean.
- The selected branch exists.

---

## Skills Demonstrated

This project demonstrates practical knowledge in:

- Jenkins Shared Libraries
- Groovy scripting
- Jenkins pipeline reuse
- CI/CD code organization
- Pipeline abstraction
- Git-based pipeline utilities
- Jenkins global library configuration
- DevOps automation practices

---

## Author

Ofri Liberman

- GitHub: https://github.com/libermanofri

---

## License

This project is intended for educational and portfolio purposes.

If you want others to reuse this project, consider adding an open-source license such as the MIT License.
