# Practice Git Security
This repo is a quick tutorial on how to scan a repo for secrets, remove them from history, and prevent leaking of secrets. Go through these steps chronologically.

## Git Leaks

1. Fork this repo. If you simply clone instead, you won't be able to push the changes you make.
2. We're going to scan our repo with `gitLeaks` (though there are [many options](https://spectralops.io/blog/top-9-git-secret-scanning-tools/)). Follow the directions [here](https://github.com/zricethezav/gitleaks) to install it.
3. Make sure that you follow the instructions to install `pre-commit` (`pip install pre-commit`), and run `pre-commit autoupdate` and `pre-commit install`
4. Think about what we want to catch. GitLeaks has an impressive list of [rules](https://github.com/zricethezav/gitleaks/blob/master/config/gitleaks.toml) that it uses to identify data it strongly suspects are secrets. It will catch secrets that follow the format of common secrets like AWS secret keys and GCP API keys, or even obscure secrets like "Etsy Access Tokens." Is there something that we want to add? (no action)
5. We will add one rule, to catch when a file contains the exact string `PASSWORD=` followed by anything. At the root of this repo, create a file called `.gitleaks.toml` with the following content:
```pre
title = "Gitleaks Custom Rules"

[extend]
useDefault = true

[[rules]]
id = "exclude PASSWORD="
description = "checks for instances of PASSWORD= followed by digits"
regex = '''PASSWORD=[0-9]{0,20}'''

[allowlist]
description = "allow README, for examples"
paths = [
    '''README.md'''
]
```

6. Now, with our default and custom rules set up, let's test that we can't commit something bad. Create a file called `password.txt` with the following content:
```AWS_KEY=AKIAIMNOJVGFDXXXE4OA```
7. Add the file `git add password.txt` and commit it `git commit -m "uh oh, secret here"`. If you get an error, congratulations! You've blocked yourself from committing a secret!
8. Yeah, let's just delete that `password.txt`. We don't want that anymore.
9. Let's direct our attention to the repo and our history. If you look at our current files, we don't see anything that raises concern. It's just this `README.md`, `.pre-commit-config.yaml`, a `docs` folder with images, and a `.gitleaks.toml` you created. But anything can happen in our history. (no action)
10. Run `git log --pretty=format:"%h%x09%an%x09%ad%x09%s"` (You can also run `git log`, but this will just be longer output, and we want something short and pretty). It should look something like the following, but longer:
```
cdf02c1 Nathaniel Larson        Fri Oct 28 03:01:33 2022 -0500  steps 5-9 added
6e82a4a Nathaniel Larson        Fri Oct 28 02:59:47 2022 -0500  steps 2-4 added
d18cd01 Nathaniel Larson        Fri Oct 28 02:48:13 2022 -0500  not suspicious
67f13d5 Nathaniel Larson        Fri Oct 28 00:34:54 2022 -0500  step 1 added
ca2dbf1 Nathaniel Larson        Fri Oct 28 00:33:55 2022 -0500  init README.md
```
11. Nothing suspicious there, right? Just kidding. One of those commits may contain a secret. We will use the `detect` command: `gitleaks detect`. It should tell us:
```
WRN leaks found: 1
```
12. Re-run, this time with the `-v` flag for a verbose output: `gitleaks detect -v`
13. Did you get an error for the `bad.env` file? It has broken our custom rule, as well as one of the default rules--but it is useful to note we only see the first (custom) rule listed in the warning. Expected output should look something like the following:
![Error using gitleaks](docs/error_gitleaks.png)
14. On to removing that secret that some bumbling idiot committed!!

## Removing Sensitive Information: Filter-Repo
*Note: Using Filter-Repo may not remove all instances of a secret. For instance, you're removing it from your fork of my repo, but the secret it still in mine! Be cognizant of this when using this in the real world*

These instructions closely follow those posted [here](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository) by GitHub.
1. Install [git-filter-repo](https://github.com/newren/git-filter-repo). On MacOS: `brew install git-filter-repo`.
2. Navigate to our repo root `cd /path/to/this/repo` (you're probably already there)
3. Identify the file we want to filter OUT: `./bad.env`
4. Check our history for the commit that we expect to filter OUT: `git log --pretty=format:"%h%x09%an%x09%ad%x09%s"`. The description is `not suspicious`.
4. Now for the powerful function: `git filter-repo --invert-paths --path bad.env` (*this is an intense function, double-check that your command is correct before running*)
5. Once we run this, let's make sure that the logs look correct. If we run `git log --pretty=format:"%h%x09%an%x09%ad%x09%s"`, most of the history should be the same, with the notable exception of our `not suspicious` commit. It's gone!
6. Let's run `gitleaks detect` to make sure. It should pass.
7. Now that we're sure that we're good. Let's run `git push origin --force --all`. This will update the remote repository. We're all fixed!

## Preventative Measures
That's nerve-wracking, and can be a bit of work! How can we just prevent these things from happening? There are many ways. Some of the most effective ways to prevent leaking of secrets is to not have them in the repository at all: (1) put secrets in a location outside of the repo, (2) use environment variables to export secrets, and then read them from those variables in the code, or (3) use a secrets management system. 

But for this small demonstration, let's say you want a `.env` file in your repo:

1. Let's prevent that specific case of secret-committing from happening. Create a `.gitignore` with the contents, which will ignore any file with the `.env` extension:
```
*.env
```
2. Actually, we can do even better than that. GitHub maintains a list of `.gitignore` templates for dozens of languages [here](https://github.com/github/gitignore/blob/main). Let's just choose the Python one [here](https://github.com/github/gitignore/blob/main/Python.gitignore). Copy the contents of that into our `.gitignore` 
3. Create a file called `bad.env`. Run `git status`. Notice how that file doesn't even show up? The power of `.gitignore`. You're safe, *as long as the `.gitignore` ignores this file!
