# Practice Git Security
This repo is a quick tutorial on how to scan a repo for secrets, remove them from history, and prevent leaking of secrets. Go through these steps chronologically. 

Any step that has "(no action)" at the end indicates that no action is required before moving to the next step of the tutorial.

## Git Leaks

1. Fork this repo. If you simply clone instead, you won't be able to push the changes you make.
2. We're going to scan our repo with `gitLeaks` (though there are [many options](https://spectralops.io/blog/top-9-git-secret-scanning-tools/)). Follow the directions [here](https://github.com/zricethezav/gitleaks) to install it.
3. Make sure that you follow the instructions to [install](https://pre-commit.com/#install) `pre-commit` (`pip install pre-commit` or `brew install pre-commit`) and run `pre-commit autoupdate`
4. In a terminal, navigate to the base of the repo `cd /path/to/this/repo`. 
5. Run `pre-commit install`. This will install the `gitLeaks` hook, because we have it specified in the file `.pre-commit-config.yaml`.
6. If your version updates, do `git add .pre-commit-config.yaml` and `git commit -m "gitLeaks version update"` to make sure the staging area is clean.
6. Think about what we want to catch. GitLeaks has an impressive list of [rules](https://github.com/zricethezav/gitleaks/blob/master/config/gitleaks.toml) that it uses to identify data it strongly suspects are secrets. It will catch secrets that follow the format of common secrets like AWS secret keys and GCP API keys, or even obscure secrets like "Etsy Access Tokens." You can also add custom formats. (no action)
7. We could run a check right now against the default rules, but we will add one rule first to catch when a file contains the exact string `PASSWORD=` followed by anything. At the root of this repo, create a file called `.gitleaks.toml` with the following content:
```pre
title = "Gitleaks Custom Rules"

[extend]
useDefault = true

[[rules]]
id = "exclude PASSWORD="
description = "checks for instances of PASSWORD= followed by digits"
regex = '''PASSWORD=.*'''

[allowlist]
description = "allow README, for examples"
paths = [
    '''README.md'''
]
```

8. Now, with our default and custom rules set up, let's test that we can't commit something bad. Create a file called `password.txt` with the following content:
```AWS_KEY=AKIAIMNOJVGFDXXXE4OA```
9. Add the file `git add password.txt`
10. Commit it: `git commit -m "uh oh, secret here"`. You should get an output like the following and NOT commit:
```
$ git commit -m "uh oh, secret here"
[INFO] Installing environment for https://github.com/zricethezav/gitleaks.
[INFO] Once installed this environment will be reused.
[INFO] This may take a few minutes...
Detect hardcoded secrets.................................................Failed
- hook id: gitleaks
- exit code: 1

○
    │╲
    │ ○
    ○ ░
    ░    gitleaks

Finding:     AWS_KEY=REDACTED
Secret:      REDACTED
RuleID:      aws-access-token
Entropy:     3.646439
File:        password.txt
Line:        1
Fingerprint: password.txt:aws-access-token:1

11:44AM INF 1 commits scanned.
11:44AM INF scan completed in 15.8ms
11:44AM WRN leaks found: 1
```
10. If you get an error, congratulations! You've blocked yourself from committing a secret! (no action)
11. Yeah, let's just delete that `password.txt`. We don't want that anymore.
12. Let's direct our attention to the repo and our history. If you look at our current files, we don't see anything that raises concern. It's just this `README.md`, `.pre-commit-config.yaml`, and a `.gitleaks.toml` you created. But BEWARE! _Anything_ could be in our history. (no action)
13. Run `git log --pretty=format:"%h%x09%an%x09%ad%x09%s"` (You can also run `git log`, but this will just be longer output, and we want something short and pretty). It should look something like the following, perhaps longer:
```
4cab3ae Nathaniel Larson        Tue Oct 8 11:56:43 2024 -0400   update GitLeaks README section
befcdeb Nathaniel Larson        Tue Oct 8 11:35:55 2024 -0400   new gitLeaks version
72d2bfe Nathaniel Larson        Tue Oct 8 11:17:23 2024 -0400   README tweak
bd300a4 Nathaniel Larson        Tue Oct 8 11:15:02 2024 -0400   scratch that
88d6706 Nathaniel Larson        Tue Oct 8 11:14:39 2024 -0400   super not sus 2024
810d24f Nathaniel               Thu Oct 12 00:05:41 2023 -0500  Merge pull request #1 from ndlarso/main
3372f89 Nathaniel Larson        Wed Oct 11 23:44:22 2023 -0500  readme instruction updates
cc84bc0 Nathaniel Larson        Wed Oct 11 23:43:05 2023 -0500  update gitleaks photo
e9e8a60 Nathaniel Larson        Wed Oct 11 23:07:04 2023 -0500  gitleaks update install and wording
ffc33c7 Nathaniel Larson        Wed Oct 11 22:55:40 2023 -0500  delete bad.env
e016876 Nathaniel Larson        Wed Oct 11 22:54:53 2023 -0500  not supicious 2023
a42c0ea Nathaniel Larson        Fri Oct 28 03:37:02 2022 -0500  Preventative Measures steps 1-3 added
cdba762 Nathaniel Larson        Fri Oct 28 03:36:09 2022 -0500  pre-commit config for repo
8b0e924 Nathaniel Larson        Fri Oct 28 03:35:51 2022 -0500  Removing Sensitive Information steps 1-7 added
0b8a5c0 Nathaniel Larson        Fri Oct 28 03:08:57 2022 -0500  steps 10-14 added, with error image
cdf02c1 Nathaniel Larson        Fri Oct 28 03:01:33 2022 -0500  steps 5-9 added
6e82a4a Nathaniel Larson        Fri Oct 28 02:59:47 2022 -0500  steps 2-4 added
d18cd01 Nathaniel Larson        Fri Oct 28 02:48:13 2022 -0500  not suspicious
67f13d5 Nathaniel Larson        Fri Oct 28 00:34:54 2022 -0500  step 1 added
ca2dbf1 Nathaniel Larson        Fri Oct 28 00:33:55 2022 -0500  init README.md
```
Note: Click `q` to exit the log

14. Nothing suspicious there, right? Just kidding. A couple of those commits may contain a secret. We will use the `detect` command. Run `gitleaks detect`. It should give us a warning ("WRN"):
```
$ gitleaks detect

    ○
    │╲
    │ ○
    ○ ░
    ░    gitleaks

11:50AM INF 15 commits scanned.
11:50AM INF scan completed in 67.6ms
11:50AM WRN leaks found: 4
```
15. Re-run, this time with the `-v` flag for a verbose output: `gitleaks detect -v`
16. Did you get an error for the `bad.env` file? It has broken our custom rule, as well as one of the default rules. Expected output should look something like the following:
```
$ gitleaks detect -v

    ○
    │╲
    │ ○
    ○ ░
    ░    gitleaks

Finding:     PASSWORD=KO6JKJJ
Secret:      PASSWORD=KO6JKJ
RuleID:      exclude PASSWORD=
Entropy:     3.373557
File:        bad.env
Line:        1
Commit:      88d6706ff229c1b016fb4496d36edd025b35b918
Author:      Nathaniel Larson
Email:       nathaniellarson@users.noreply.github.com
Date:        2024-10-08T15:14:39Z
Fingerprint: 88d6706ff229c1b016fb4496d36edd025b35b918:bad.env:exclude PASSWORD=:1

Finding:     PASSWORD=EVU7566
Secret:      PASSWORD=EVU756
RuleID:      exclude PASSWORD=
Entropy:     3.773557
File:        bad.env
Line:        1
Commit:      e016876278b5f459896cbd052e9332d22616cde1
Author:      Nathaniel Larson
Email:       nathaniellarson@users.noreply.github.com
Date:        2023-10-12T03:54:53Z
Fingerprint: e016876278b5f459896cbd052e9332d22616cde1:bad.env:exclude PASSWORD=:1

Finding:     ...=55300
FAKE_AWS_KEY=AKIAIMNOJVGFDXXXE4OAA
Secret:      AKIAIMNOJVGFDXXXE4OA
RuleID:      aws-access-token
Entropy:     3.646439
File:        bad.env
Line:        2
Commit:      d18cd01613d38d197cdaed009b5e008107b13f6a
Author:      Nathaniel Larson
Email:       nathaniellarson@users.noreply.github.com
Date:        2022-10-28T07:48:13Z
Fingerprint: d18cd01613d38d197cdaed009b5e008107b13f6a:bad.env:aws-access-token:2

Finding:     PASSWORD=553000
Secret:      PASSWORD=55300
RuleID:      exclude PASSWORD=
Entropy:     3.378783
File:        bad.env
Line:        1
Commit:      d18cd01613d38d197cdaed009b5e008107b13f6a
Author:      Nathaniel Larson
Email:       nathaniellarson@users.noreply.github.com
Date:        2022-10-28T07:48:13Z
Fingerprint: d18cd01613d38d197cdaed009b5e008107b13f6a:bad.env:exclude PASSWORD=:1

11:52AM INF 15 commits scanned.
11:52AM INF scan completed in 69.7ms
11:52AM WRN leaks found: 4
```
16. We can inspect these commits more closely to make sure there's nothing we are missing. Run `git show <commit-id>` (e.g. `git show 88d6706ff229c1b016fb4496d36edd025b35b918` or even shorter `git show 88d6706`) and look at the changes added to the file. You can also do this from the GitHub UI.
17. What's next? Removing that secret that some bumbling idiot committed!!

## Removing Sensitive Information: BFG
*Note: `bfg` may not remove all instances of a secret. For instance, you're removing it from your fork of my repo, but the secret it still in mine! Be cognizant of this when using this in the real world*

These instructions closely follow those posted [here](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository) by GitHub.
1. Install [bfg](https://rtyley.github.io/bfg-repo-cleaner/). On MacOS: `brew install bfg`.
2. Navigate to our repo root `cd /path/to/this/repo` (you're probably already there)
3. Identify the file we want to filter OUT: `bad.env`. We can either remove this file completely (using the `--delete-files` flag) or replace all the text of particular files (using the `--replace-text` flag). We will remove `bad.env` completely.
4. In this case, we will check our history for the commit(s) that we expect to filter OUT: `git log --pretty=format:"%h%x09%an%x09%ad%x09%s"`. The descriptions of the three commits involving `bad.env` are `not suspicious`,  `not suspicious 2023`, and `super not sus 2024`. 
4. Now for the powerful function: `bfg --delete-files bad.env` (*this is an intense function, double-check that your command is correct before running*)
5. Once we run this, let's make sure that the logs look correct. If we run `git log --pretty=format:"%h%x09%an%x09%ad%x09%s"`, most of the history should be the same, with the notable exception of our previously mentioned commits. Run `git show 88d6706`, and notice that there are no changes in the commit anymore! It's simply empty, because before it was only adding `bad.env`. If you do the same with the other two commits, you'll see `bad.env` doesn't show up at all anymore!
6. Let's run `gitleaks detect` to make sure. It should pass.
```
$ gitleaks detect

    ○
    │╲
    │ ○
    ○ ░
    ░    gitleaks

12:36PM INF 12 commits scanned.
12:36PM INF scan completed in 65.8ms
12:36PM INF no leaks found
```
6. STOP. In a real world scenario, do extensive testing to ensure that your repo is how you want it to be. Then continue (no action)
7. Now that we're sure that we're good. Let's run `git push --force`. This will update the remote repository. We're all fixed!
8. Feel free to go to your fork of the repo to see that `bad.env` is not in the remote history. NOTE: in a real-world scenario, make sure that all contributors to the project pull fresh versions of the repo and delete old repositories.

## Preventative Measures
That's nerve-wracking, and can be a bit of work! How can we just prevent these things from happening? There are many ways. Some of the most effective ways to prevent leaking of secrets is to not have them in the repository at all: 

1. put secrets in a location outside of the repo,
2. use environment variables to export secrets, and then read them from those variables in the code, or 
3. use a secrets management system. 

But for this small demonstration, let's say you want a `.env` file in your repo:

1. Let's prevent that specific case of secret-committing from happening. Create a `.gitignore` with the contents, which will ignore any file with the `.env` extension:
```
*.env
```
2. Actually, we can do even better than that. GitHub maintains a list of `.gitignore` templates for dozens of languages [here](https://github.com/github/gitignore/blob/main). Let's just choose the Python one [here](https://github.com/github/gitignore/blob/main/Python.gitignore). Copy the contents of that into our `.gitignore` 
3. Create a file called `bad.env`. Run `git status`. Notice how that file doesn't even show up? The power of `.gitignore`. You're safe, *as long as the `.gitignore` ignores this file!

## Do Even Better: SeCureLI

The [seCureLI](https://github.com/slalombuild/secureli) open source project that not only scans your local repo for secrets before you commit code to a remote repository like gitLeaks, but also installs linters based on the code of your project to support security and coding best practices, and configures all the hooks needed so you don’t have to. It collects valuable tools based on the type of project you're working on. 

Visit the [seCureLI](https://github.com/slalombuild/secureli) page to get started with this tool.
