# DevOps Learning Project --- Jira, GitHub, Docker Compose and Jenkins

This README records the practical workflow for the DevOps learning
project: tracking work in Jira, managing source code with Git/GitHub,
containerizing two static websites with Docker, and preparing a Jenkins
CI/CD pipeline.

> **Project status:** Jira/GitHub workflow and Docker Compose have been
> worked through. Jenkins is documented as the next stage; do not treat
> Jenkins as already installed or configured.

## 1. Project overview

-   **Working directory (Ubuntu on WSL 2):** `~/devops`
-   **GitHub repository:** `ndikubwimana12/devops`
-   **Jira project:** `Devops`
-   **Jira issues discussed:** `DEV-1`, `DEV-3`, `DEV-4`, `DEV-5`
-   **Application:** two static websites served by Nginx in separate
    containers
-   **Main website port:** `8080`
-   **Course website port:** `8081`
-   **Docker engine:** Docker Desktop using its WSL 2 backend

The intended workflow is:

``` text
Jira task
   |
   v
Git feature branch
   |
   v
Edit files -> test locally
   |
   v
Commit with Jira issue key
   |
   v
Push branch to GitHub
   |
   v
Pull Request -> review -> merge into main
   |
   v
Jira issue updated
   |
   v
Jenkins pipeline (next stage)
   |
   v
Validate -> build -> deploy
```

## 2. Jira setup and task tracking

Create a Jira project for the learning work (the project used here is
named **Devops**).

### Recommended issue workflow

Use the board statuses:

-   **To Do** --- work has not started.
-   **In Progress** --- work is being done.
-   **Done** --- work is completed and accepted.

Each task should have a clear title, description, and acceptance
criteria. Example tasks discussed:

  Jira issue   Example task
  ------------ ------------------------------------------
  `DEV-1`      Create and configure the project
  `DEV-3`      Create the home page (`index.html`)
  `DEV-4`      Create the about page (`about.html`)
  `DEV-5`      Create the contact page (`contact.html`)

The issue list and statuses can change; check the Jira board for the
current state.

### Jira working practice

1.  Select one issue from **To Do**.
2.  Move it to **In Progress**.
3.  Create a Git branch named with the issue key.
4.  Make the changes and test them.
5.  Commit and push the branch.
6.  Open a Pull Request (PR) and merge it after review.
7.  Move the Jira issue to **Done** when its acceptance criteria are
    met.

### Link Jira with GitHub

Use the GitHub for Jira integration/app in Jira to connect the Jira site
to the GitHub account or organization that owns the repository.
Authorize the requested access and select the repository if prompted.

Include the Jira issue key in branch names, commit messages, and PR
titles. This helps Jira associate development activity with the issue.
The integration must be installed and authorized before this automatic
linking can work.

## 3. Git and GitHub workflow

### Check the repository

Run commands from the project directory:

``` bash
cd ~/devops
git status
git remote -v
git branch
```

### Configure the GitHub remote with SSH

The repository used in this learning project is:

``` text
git@github.com:ndikubwimana12/devops.git
```

Set the remote:

``` bash
git remote set-url origin git@github.com:ndikubwimana12/devops.git
```

Check it:

``` bash
git remote -v
```

If SSH authentication has not been configured, create an SSH key, add
its public key to GitHub, and test the connection:

``` bash
ssh -T git@github.com
```

Do not share your private SSH key or access tokens. GitHub account
passwords are not accepted for Git operations over HTTPS; use SSH or a
personal access token.

### Start work on a Jira issue

First update the local `main` branch:

``` bash
git switch main
git pull origin main
```

Create a feature branch. Example:

``` bash
git switch -c feature/DEV-3-index-html
```

Other examples:

``` bash
git switch -c feature/DEV-4-about-page
git switch -c feature/DEV-5-contact-page
```

Use the branch that matches the issue you are actually working on. A
branch name is only an example until you create it.

### Save and push changes

Check changed files:

``` bash
git status
```

Stage and commit:

``` bash
git add .
git commit -m "DEV-3: Create home page"
```

Push the current branch:

``` bash
git push -u origin feature/DEV-3-index-html
```

Replace the branch name and issue key with the ones you are actually
using.

Useful commands:

``` bash
git diff                 # Review unstaged changes
git diff --staged        # Review staged changes
git log --oneline -5     # View recent commits
git branch               # List local branches
git branch -a            # List local and remote branches
```

If Git reports `nothing to commit, working tree clean`, there are no new
changes staged or available to commit. Check that you edited and saved
the intended files.

### Create and merge a Pull Request

1.  Push your feature branch to GitHub.
2.  Open the repository on GitHub.
3.  Create a Pull Request from your feature branch into `main`.
4.  Review the changes and resolve any conflicts.
5.  Merge the PR when it is ready.
6.  Update the Jira issue.

If merging locally is required instead of using a PR:

``` bash
git switch main
git pull origin main
git merge feature/DEV-3-index-html
git push origin main
```

Prefer a Pull Request for practice and review.

### Delete a feature branch after merging

Delete the local branch:

``` bash
git branch -d feature/DEV-3-index-html
```

Delete the remote branch:

``` bash
git push origin --delete feature/DEV-3-index-html
```

Only delete a branch after confirming that its changes have been merged
or are no longer needed.

## 4. Docker Desktop and WSL 2 setup

Docker Desktop is installed on Windows and configured to use the **WSL 2
backend**.

In Docker Desktop:

1.  Open **Settings → General** and confirm the WSL 2 backend is
    selected.
2.  Open **Settings → Resources → WSL Integration**.
3.  Enable integration for the Ubuntu distribution.
4.  Apply the settings and restart Docker Desktop if requested.

If WSL integration settings change, run this in Windows PowerShell:

``` powershell
wsl --shutdown
```

Then reopen Ubuntu.

### Verify Docker from Ubuntu

``` bash
docker --version
docker compose version
docker version
docker context ls
```

`docker version` should show both a Client and a Server. In this setup
the server should identify Docker Desktop.

The Docker context list may include `default` and `desktop-linux`. Since
the current connection already reaches the Docker Desktop server, do not
change contexts unless a connection problem requires it.

## 5. Dockerfile for the main static website

For a plain HTML/Tailwind website, Node.js is not required to serve the
built/static files. Nginx can serve the website.

Example `Dockerfile` in the main website directory:

``` dockerfile
FROM nginx:alpine

COPY . /usr/share/nginx/html

EXPOSE 80
```

This copies the website files into Nginx's default web root and exposes
port 80 inside the container.

Example `.dockerignore`:

``` text
.git
.gitignore
README.md
```

Do not add the `course` directory to `.dockerignore` if you need it for
the second website or build context.

### Build and run the main website by itself

``` bash
docker build -t devops-web:v1 .
docker run -d --name devops-web -p 8080:80 devops-web:v1
```

Open:

``` text
http://localhost:8080
```

Useful container commands:

``` bash
docker ps
docker ps -a
docker logs devops-web
docker stop devops-web
docker start devops-web
docker rm -f devops-web
docker images
```

If a container name is already in use, remove the old container before
creating another container with the same name.

## 6. Docker Compose for two websites

The project uses Docker Compose to manage two website containers
together:

-   `web`: builds the main website from the current directory and maps
    host port `8080` to container port `80`.
-   `course`: uses the `nginx:alpine` image, mounts the local `course`
    directory, and maps host port `8081` to container port `80`.

Example project layout:

``` text
devops/
├── Dockerfile
├── compose.yaml
├── index.html
├── about.html
├── contact.html
└── course/
    └── index.html
```

Your actual project may contain additional files and folders.

### Correct `compose.yaml`

``` yaml
services:
  web:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: devops-web
    ports:
      - "8080:80"
    restart: unless-stopped

  course:
    image: nginx:alpine
    container_name: course-web
    volumes:
      - ./course:/usr/share/nginx/html:ro
    ports:
      - "8081:80"
    restart: unless-stopped
```

Important YAML rules:

-   Indentation matters. Use consistent spaces (commonly two spaces per
    level).
-   Put each port mapping on a new list line below `ports:`.
-   Write `context`, not `contex`.
-   Do not write `ports: -"8080:80"`; write `ports:` followed by an
    indented `- "8080:80"`.

### Validate, build, start and inspect

Run these commands in the directory containing `compose.yaml`:

``` bash
docker compose config
docker compose up -d --build
docker compose ps
```

Open the websites:

-   Main website: <http://localhost:8080>
-   Course website: <http://localhost:8081>

More Compose commands:

``` bash
docker compose logs
docker compose logs -f
docker compose logs web
docker compose logs course
docker compose restart
docker compose stop
docker compose start
docker compose down
docker compose down --rmi local
```

`docker compose down` stops and removes the Compose containers and
network. It does not normally delete the source files in your project
directory. Use `--rmi local` only when you also want to remove locally
built images for this Compose project.

When website source files change, rebuild and recreate as needed:

``` bash
docker compose up -d --build
```

The `course` service uses a bind mount, so changes in its local
`course/` directory are served without rebuilding the Nginx image.

## 7. Troubleshooting notes

### `docker: unknown command: docker compose`

Docker Compose is not available to the Docker CLI being used. Check
Docker Desktop WSL integration and run:

``` bash
docker compose version
```

Do not install multiple Docker engines or remove packages without first
checking which Docker CLI and server are active.

### `services.web.build additional properties 'contex' not allowed`

The YAML property was misspelled. Use:

``` yaml
build:
  context: .
```

### Compose says `services must be an array` or reports a YAML error

Check the indentation and list formatting in `compose.yaml`. The
top-level `services:` key should contain service mappings such as `web:`
and `course:`. Port mappings must be formatted as lists:

``` yaml
ports:
  - "8080:80"
```

Validate before starting:

``` bash
docker compose config
```

### Port is already allocated

Another process or container may already be using the host port. Check
running containers:

``` bash
docker ps
```

Stop or remove the conflicting container, or choose another unused host
port.

### Website does not load

Check container status and logs:

``` bash
docker compose ps
docker compose logs web
docker compose logs course
```

Confirm that the main website files are in the build context and that
`course/index.html` exists for the course website.

## 8. Jenkins --- next learning stage

**Jenkins setup has not yet been confirmed as completed.** The following
is a starting checklist for adding Jenkins after the Docker Compose
project is working.

### What Jenkins will do

Jenkins is an automation server. A pipeline can automatically:

1.  Check out the GitHub repository.
2.  Validate the Compose configuration.
3.  Build the website image(s).
4.  Run checks.
5.  Deploy or restart the containers on an approved environment.

### Start Jenkins with Docker

A simple local Jenkins controller can be started with the official LTS
image:

``` bash
docker volume create jenkins_home

docker run -d \
  --name jenkins \
  --restart unless-stopped \
  -p 8082:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts
```

Open Jenkins:

``` text
http://localhost:8082
```

Retrieve the initial administrator password:

``` bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Copy the password into the Jenkins setup page. Then install the
suggested plugins and create an administrator account. Keep the password
private.

Useful Jenkins container commands:

``` bash
docker ps
docker logs jenkins
docker logs -f jenkins
docker restart jenkins
docker stop jenkins
docker start jenkins
```

### Configure a Jenkins Pipeline job

1.  In Jenkins, select **New Item**.
2.  Give the job a name, for example `devops-website-pipeline`.
3.  Select **Pipeline** and create the job.
4.  Configure the GitHub repository and credentials if using a
    repository-based pipeline.
5.  Add a `Jenkinsfile` to the root of the Git repository.
6.  Configure the job to use **Pipeline script from SCM**, choose Git,
    enter the repository URL, select the branch, and set the script path
    to `Jenkinsfile`.
7.  Save and run **Build Now**.

For a private GitHub repository, configure credentials in Jenkins rather
than putting passwords or tokens in the Jenkinsfile.

### Starter Jenkinsfile: validate and build

This starter pipeline assumes the Jenkins agent has Git, Docker CLI, and
the Docker Compose plugin installed and configured to access a Docker
engine. A Jenkins container does not automatically have access to the
host's Docker engine just because Docker Desktop is installed. Configure
a suitable Jenkins agent or secure Docker access before expecting Docker
commands to work.

Create a file named `Jenkinsfile` in the repository root:

``` groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate Compose') {
            steps {
                sh 'docker compose config'
            }
        }

        stage('Build Website Images') {
            steps {
                sh 'docker compose build'
            }
        }
    }
}
```

This pipeline validates the Compose file and builds the images. It does
not deploy the containers. Add deployment only after the Jenkins agent
and deployment environment have been configured and tested.

### Connect GitHub events to Jenkins

For automatic builds after a push or Pull Request, configure a GitHub
webhook to the Jenkins endpoint and enable the matching trigger in the
Jenkins job. This requires Jenkins to be reachable by GitHub; a Jenkins
instance available only at `localhost` is not reachable from GitHub's
servers. For local learning, a manual **Build Now** is sufficient until
webhook networking is configured.

## 9. Recommended daily workflow

For each Jira task:

``` bash
cd ~/devops
git switch main
git pull origin main
git switch -c feature/DEV-<issue>-<short-description>
```

Make and test the change, then:

``` bash
git status
git add .
git commit -m "DEV-<issue>: Describe the change"
git push -u origin feature/DEV-<issue>-<short-description>
```

Open a Pull Request on GitHub, review and merge it, then update Jira.

For Docker Compose changes:

``` bash
docker compose config
docker compose up -d --build
docker compose ps
```

When Jenkins is configured, use the pipeline to automate the validation
and build steps rather than relying only on manual commands.

## 10. Important safety practices

-   Never commit passwords, SSH private keys, API tokens, or Jenkins
    credentials.
-   Use Jenkins credentials management for secrets.
-   Do not expose the Docker daemon over an unauthenticated TCP
    connection.
-   Be careful when mounting the Docker socket into a Jenkins container:
    access to the Docker daemon can grant powerful control over the
    host.
-   Use Pull Requests and review changes before merging into `main`.
-   Keep Jira issue keys in branch names, commits, and PR titles.
-   Verify the target environment before running deployment commands.

## 11. Quick command reference

  Purpose                            Command
  ---------------------------------- -----------------------------------------------
  Check Git changes                  `git status`
  Update main                        `git pull origin main`
  Create a feature branch            `git switch -c feature/DEV-3-index-html`
  Stage changes                      `git add .`
  Commit with Jira key               `git commit -m "DEV-3: Create home page"`
  Push branch                        `git push -u origin feature/DEV-3-index-html`
  Check Docker                       `docker version`
  Check Compose                      `docker compose version`
  Validate Compose file              `docker compose config`
  Build and start services           `docker compose up -d --build`
  List Compose services              `docker compose ps`
  Follow service logs                `docker compose logs -f`
  Stop and remove Compose services   `docker compose down`
  View Jenkins logs                  `docker logs -f jenkins`

------------------------------------------------------------------------

**End of learning notes.** Update this README as each new stage is
completed, especially when Jenkins agents, GitHub webhooks, automated
tests, and deployment are configured.
