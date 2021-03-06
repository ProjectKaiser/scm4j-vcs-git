[![Release](https://jitpack.io/v/scm4j/scm4j-vcs-git.svg)](https://jitpack.io/#scm4j/scm4j-vcs-git)
[![Build Status](https://travis-ci.org/scm4j/scm4j-vcs-git.svg?branch=master)](https://travis-ci.org/scm4j/scm4j-vcs-git)
[![Coverage Status](https://coveralls.io/repos/github/scm4j/scm4j-vcs-git/badge.svg?branch=master)](https://coveralls.io/github/scm4j/scm4j-vcs-git?branch=master)

# Overview
scm4j-vcs-git is lightweight library for execute basic Git VCS operations (merge, branch create etc). It uses [scm4j-vcs-api](https://github.com/scm4j/scm4j-vcs-api) exposing IVCS implementation for Git repositories and [JGit](https://eclipse.org/jgit/) as framework to work with Git repositories.
Features:
- Working wit branches: create, remove, browse
- Branch merge with result return (success or list of conflicted files)
- Summarized diff between branches
- File content getting and setting
- File create and remove
- Working with tags: create, remove, browse

Use cases
- VCS server hooks
- Build machines
  - checking in\out, tagging
- Software project management systems
  - Create own branches from GUI, browse commits, product versions management, etc
- Product release automation
  - automatic merging, forking, tagging, version bumping, etc
  - Example: [scm4j-releaser](https://github.com/scm4j/scm4j-releaser)


# Terms
- Workspace Home
  - Home local folder of all folders used by vcs-related operations. See [scm4j-vcs-api](https://github.com/scm4j/scm4j-vcs-api) for details
- Locked Working Copy, LWC
  - Local folder where vcs-related operations are executed. Provides thread- and process-safe repository of working folders. See [scm4j-vcs-api](https://github.com/scm4j/scm4j-vcs-api) for details
- Test Repository
  - Git repository which is used to execute functional tests
  - File-based repository is used
  - Generates new one before and deletes after each test
  - Named randomly (uuid is used) 

# Using scm4j-vcs-git
- Add github-hosted scm4j-vcs-git project as maven dependency using [jitpack.io](https://jitpack.io/). As an example, add following to gradle.build file:
	```gradle
	allprojects {
		repositories {
			maven { url "https://jitpack.io" }
		}
	}
	
	dependencies {
	 	// versioning: master-SNAPSHOT (lastest build, unstable), + (lastest release, stable) or certain version (e.g. 1.1)
		compile 'com.github.scm4j:scm4j-vcs-git:+'
	}
	```
	Or download release jars from https://github.com/scm4j/scm4j-vcs-git/releases
- Code snippet
	```java
	final String WORKSPACE_DIR = System.getProperty("java.io.tmpdir") + "git-workspaces";
	IVCSWorkspace workspace = new VCSWorkspace(WORKSPACE_DIR);
	String repoUrl = "https://github.com/MyUser/MyRepo";
	IVCSRepositoryWorkspace repoWorkspace = workspace.getVCSRepositoryWorkspace(repoUrl);
	IVCS vcs = new GitVCS(repoWorkspace);
	vcs.setCredentials("user", "password"); // if necessary
	```
- Use methods of `IVCS` interface. See [scm4j-vcs-api](https://github.com/scm4j/scm4j-vcs-api) for details
- Use `vcs.setProxy()` and `vcs.setCredentials()` if necessary
- Use `VCSTag createUnannotatedTag(String branchName, String tagName, String revisionToTag)` to create git unannontated tag with name `tagName` on `revisionToTag` commit of branch `branchName`. If `branchName` is null then master branch is used. If `revisionToTag` is null then head of branch `branchName` is used.

# Implementation details
- [JGit](https://eclipse.org/jgit/) is used as framework to work with Git repositories
- Each vcs operation is executed within a LWC
- `getLocalGit(IVCSLockedWorkingCopy wc)` method is used to create a Git implementation to execute vcs operations within `wc` Working Copy
  - If provided LWC is empty then current Test Repository is cloned into this LWC, otherwise existing repository is just switched to the required branch
- If `IVCS.setProxy()` is called then provided proxy is used for each url which contains `repoUrl`

# Functional testing
- New local file-based Test Repository is created before each test and deletes automatically after each test
- To execute tests just run GitVCSTest class as JUnit test. Tests from VCSAbstractTest class will be executed. See [scm4j-vcs-test](https://github.com/scm4j/scm4j-vcs-test) for details
- Or run `gradle test` to execute tests

# Limitations
- Commit messages can not be attached to branch create and delete operations because Git does not expose these operations as separate commits
