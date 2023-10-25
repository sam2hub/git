<!-- TOC -->

- [Setting Up GitHub multiple branch workflow](#setting-up-github-multiple-branch-workflow)
    - [Branches](#branches)
        - [Main Branches](#main-branches)
        - [Topic Branches](#topic-branches)
    - [Initial setup of new repos](#initial-setup-of-new-repos)
        - [In GitHub](#in-github)
    - [Populate the repo](#populate-the-repo)
        - [To populate the repo with a pre-populated cookiecutter Ansible role:](#to-populate-the-repo-with-a-pre-populated-cookiecutter-ansible-role)
        - [To clone the repo to your local working area:](#to-clone-the-repo-to-your-local-working-area)
    - [Once the master branch is in place:](#once-the-master-branch-is-in-place)
        - [In Jenkins](#in-jenkins)
    - [Development and testing](#development-and-testing)
        - [If Developing Ansible Roles and/or Playbooks...](#if-developing-ansible-roles-andor-playbooks)
        - [Avoiding Conflicts](#avoiding-conflicts)
        - [Topic (Feature) branch creation](#topic-feature-branch-creation)
        - [Merging into `develop`](#merging-into-develop)
        - [Merging `develop` into `master`](#merging-develop-into-master)
        - [Tagging the new release with semantic version](#tagging-the-new-release-with-semantic-version)
    - [Appendices](#appendices)
        - [xyxyxy IS Git Workflow graphic](#xyxyxy-is-git-workflow-graphic)

<!-- /TOC -->

# Setting Up GitHub multiple branch workflow

The following document describes the branching workflow that is being applied
to the git SCM in use at xyxyxy.

## Branches

### Main Branches

- `master` - Master is the production branch that is deemed stable.
    Releases (tags) will be cut from this branch. Additional Details on Release (tags)
    Specifications can be found [here](https://xyxyxygithub.xyxyxyglobal.com/INFRA/documentation/blob/master/isprojects/product_feature_version_specs.md)

- `develop` - Successfully-tested "topic" branches will be merged into
    develop for complete end-to-end testing.

### Topic Branches

Topic branches, also known as "feature" or "fix" branches, are where actual
development is done. These branches can be tested in Jenkins with multibranch
jobs, along with pull requests into the `develop` branch.

## Initial setup of new repos

### In GitHub

Navigate to the appropriate organization in GitHub; the Infrastructure Services
team generally uses the [INFRA](https://xyxyxygithub.xyxyxyglobal.com/INFRA)
organization.  Click "New".

Give the repo an appropriate name and set permissions as needed.
In many cases, particularly for Ansible roles, it's necessary to
make the repo "public", but if you're making one for your own user
you can leave it private if you so choose.

![CreateNewRepo](images/CreateNewRepo.PNG)

## Populate the repo
Once you've created the repo you can populate it with a Cookiecutter Ansible role or, if not creating the repo for an ansible role, clone it to wherever you do your
editing.

Important note: If you are creating a new Ansible role,
you will want to leverage the Cookiecutter tool to prepopulate the
role with all the tools needed to develop and test locally.

See the [Cookiecutter for new Ansible Repos](#cookiecutter-for-new-ansible-repos)
section for more details.

### To populate the repo with a pre-populated cookiecutter Ansible role:
- Follow the [procedures](https://xyxyxygithub.xyxyxyglobal.com/INFRA/cookiecutter-ansible) in the Cookiecutter repo to create the ansible role.
- Then run the following steps to push the Cookiecutter role to the new repo you created.  These steps will
  initialize the local cookiecutter role as a git repo, add all files to a commit call "initial commit" and
  then push the local repo to the newly created repo you created above.

```bash
cd cookiecutter-role-folder
git remote add origin https://xyxyxygithub.xyxyxyglobal.com/INFRA/your-newly-created-repo-name.git
git init
git add .
git commit -m "initial commit"
git push -u origin master
```

### To clone the repo to your local working area:

```bash
$ git clone git@xyxyxygithub.xyxyxyglobal.com:INFRA/ansible-role_linux_oracle.git
    Cloning into 'ansible-role_linux_oracle'...
    Enter passphrase for key '/c/Users/<user>/.ssh/id_rsa':
    Enter passphrase for key '/c/Users/<user>/.ssh/id_rsa-gitonly':
    warning: You appear to have cloned an empty repository.
```

There are additional settings that need to be done in the repo,
but unfortunately some of them won't be available until there's
code in a master branch.  Simply edit a README.md file and then
push it up to the master branch.

```bash
vi README.md
<type in some useful info>
git add README.md
git commit -am "updated the README.md file"
git push origin master
```

## Once the master branch is in place:

- Create a new develop branch: click "Branch: master" and
    type "develop"into the field, then click "Create branch: develop"

![CreateDevelopBranch](images/CreateDevelopBranch.PNG)

- Set the develop branch as the default: Click "Settings",
    then click "Branches".  Click "master" and set it to "develop".  Click Update.

![SetDefaultBranch](images/SetDefaultBranch.PNG)

- Protect the master branch: click "Choose a branch" and select "master".
    Click the "Protect this branch" checkbox, and then check each of the four
    checkboxes that become available:
    - Require pull request reviews before merging
        - Dismiss stale pull request approvals when new commits are pushed
    - Require status checks to pass before merging
    - Restrict who can push to this branch
    - Include administrators

Click "Save Changes".

![ProtectMasterBranch](images/ProtectMasterBranch.PNG)

- Add the "svcinfrabot" user to the collaborators: Click "Collaborators and Teams".
    At the bottom, under "Search by username, full name or email address",
    type "svcinfrabot", and then click "Add Collaborator".

![AddSvcInfraBot](images/AddSvcInfraBot.PNG)

- Add Jenkins integration: Click "Integrations and Services".
    Click "Add service".  Search for "Jenkins" and select the "Jenkins GitHub" plugin.
    Insert this URL into the "Jenkins hook url": `https://jenkins.xyxyxyglobal.com/github-webhook/`
    Make sure the "Active" checkbox is selected and click "Add service".

![AddJenkinsIntegration](images/AddJenkinsIntegration.PNG)

- Add Jenkins Github webhook: Click "Hooks".  Click "Add webhook".
    The Payload URL is the same as the Jenkins integration URL:
    `https://jenkins.xyxyxyglobal.com/github-webhook/`

Click the radio button for "Let me select individual events".
Make sure the checkboxes for "Push" and "Pull request" are selected.
Click "Add webhook".

![AddJenkinsWebhook](images/AddJenkinsWebhook.PNG)

Once this is done, you'll be able to create a topic branch in your
development environment, push that branch to the repo, and submit pull
requests against the develop branch.

### In Jenkins

In order to properly test your code, you'll need to set up a multibranch
job in Jenkins.  The good news is that you can simply clone an existing
multibranch job, and then point it to the Jenkinsfile you'll develop
(or let cookiecutter create).  From within the area in Jenkins you want
to add the new job, click "New Item."  Then fill in a name, generally the
same one as your new repo.  At the bottom, you can provide the name of an
existing repo you want to clone, or select "Multibranch Pipeline".  Click OK.

![CreateJenkinsMBJob](images/CreateJenkinsMBJob.PNG)

Then you just need point the job (in the "Repository" field under "Branch Sources) to the appropriate repo that you just created, and click Save.

![SetRepoInMBConfig](images/SetRepoInMBConfig.PNG)

It will attempt to scan the GitHub repo for branches and pull them in for testing.

## Development and testing

### If Developing Ansible Roles and/or Playbooks...

When creating new Ansible roles, a tool named Cookiecutter should be used.
After creating the new repo, but before cloning it to wherever you do your
development, use cookiecutter to populate the new role with the appropriate
file structure and testing tools.

See [Ansible Getting Started on Windows Desktops with WSL](https://xyxyxygithub.xyxyxyglobal.com/INFRA/documentation/blob/master/ansible/ansible-getting-started.md)
Documentation on configuring workstation for Ansible Development on Windows Machines.

For instructions on installing and using cookiecutter ansible template, See:
[https://xyxyxygithub.xyxyxyglobal.com/INFRA/cookiecutter-ansible](https://xyxyxygithub.xyxyxyglobal.com/INFRA/cookiecutter-ansible)

### Avoiding Conflicts

In order to avoid merge conflicts, and out of sync branches, running the
following commands will keep the local working copy in sync. It is advised
to run these command frequently, especially at the start of a work day:

```shell
# Switch to the develop branch
$ git checkout develop

# Rebase the develop branch
$ git rebase develop

# Fetch changes from the upstream develop branch into our working copy
$ git pull upstream develop
```

Once the develop branch is up-to-date, you can create a new topic branch and
begin your development in it.

### Topic (Feature) branch creation

```shell
# Create a new topic branch
$ git checkout -b newfix

<write some code and locally test it>

# Add your new files
$ git add .

# Commit the changes to the local branch and repo
$ git commit -am 'detailed commit msg'

# Push the updated topic branch to INFRA
$ git push origin newfix
```

Once the branch is pushed into GitHub, if things are configured correctly,
Jenkins will attempt to test the topic branch.

![TopicBranchTesting](images/TopicBranchTesting.PNG)

If it's successful, you can submit a pull request to get your changes merged
into the `develop` branch.

### Merging into `develop`

After you've pushed your topic branch to the repo, it should show up as a
recently updated branch.

![RecentlyPushedBranches](images/RecentlyPushedBranches.PNG)

Click "Compare & pull request" to begin a PR.  Make sure that it lists "base"
as 'develop' and "compare" as your topic branch.
Put some description in the title, and add a reviewer (or many) to review your PR.
Click "Create pull request."

![OpenAPullRequest](images/OpenAPullRequest.PNG)

If everything is configured correctly, at this point Jenkins should
pull in a pseudo-commit that is the result of applying your merge to
the develop branch, and test it.  If it is successful, it will
report this back to GitHub.  Once a reviewer has reviewed the code,
it can be merged.  At that point, the `develop` branch will be retested.

### Merging `develop` into `master`

Once you're satisfied that the `develop` branch is stable and you're ready
to start using those updates in production, you'll submit a pull request
to merge `develop` into `master`.  Click "New pull request".  By default,
it will set "base" and "compare" to 'develop', so change "base" to `master`.
Put in a relevant title to describe your updates, add reviewers,
and click "Create pull request".

Jenkins should detect the updated repo and attempt to test the PR
and relevant branches.  Because `master` is a protected branch, merging
won't be possible until the tests are successful, and at least one reviewer
has approved the changes.  Once that is done, you can merge into the master
branch, and the code will be available for production use as a new "release".

### Tagging the new release with semantic version

In many situations, it's necessary to "tag" the new release before it can be used.

Additional Details on Release (tags) Specifications can be found [here](https://xyxyxygithub.xyxyxyglobal.com/INFRA/documentation/blob/master/isprojects/product_feature_version_specs.md)

For example, in our Ansible Tower environment, playbooks pull roles from Git based
on their semantic version, like this role from the `ansible-linux_base` playbook
that gets run on new virtual machine builds:

```bash
- name: ansible-role_satellite
  src: https://xyxyxygithub.xyxyxyglobal.com/INFRA/ansible-role_satellite
  scm: git
  version: 1.0.3
```

Because the `version` is set to `1.0.3`, which points to a specific commit,
simply updating the master branch with new code won't change what that role
does when it's called by the playbook.

We use `semantic versioning` for our tags, which follows a simple
`major.minor.patch` format.
The numbering starts at `0.1.0` for testing pre-releases; the first stable
release will be `1.0.0`.
Each number gets incremented by 1 as needed:

- `patch` - Incremented for small enhancements or bugfixes.
- `minor` - Incremented for more substantial, but backwards-compatible, enhancements.
- `major` - Incremented for major updates that may break existing uses of the code.

The easiest way to add a tag to your updated code is to pull the master branch
down to your development environment, add the tag, and push the tag back up to
the repo:

```bash
git checkout master
git pull origin master
git tag -a "1.0.4" -m "fixed satellite channel keys, see issue 113"
git push origin 1.0.4
```

Then you can update any code that pulls the repo from GitHub based on the tag.
The tag will be visible in the repo by going to the main repo page,
clicking `Branch: develop`, and selecting the "tags" tab.

![ViewingRepoTags](images/ViewingRepoTags.PNG)

## Appendices

### xyxyxy IS Git Workflow graphic

A handy visualization PDF of the Git workflow we use is located here:
[xyxyxy IS Git Workflow](images/xyxyxy-IS-Git-Workflow.pdf)
