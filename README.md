# Practice Git Security
This repo is a quick tutorial on how to scan a repo for secrets, remove them from history, and prevent leaking of secrets. Go through these steps chronologically.

## Git Leaks

1. Fork this repo. If you simply clone instead, you won't be able to push the changes you make.
2. We're going to scan our repo with `gitLeaks` (though there are [many options](https://spectralops.io/blog/top-9-git-secret-scanning-tools/)). Follow the directions [here](https://github.com/zricethezav/gitleaks) to install it.
3. Make sure that you follow the instructions to install `pre-commit` (`pip install pre-commit`), and run `pre-commit autoupdate` and `pre-commit install`
4. Think about what we want to catch. GitLeaks has an impressive list of [rules](https://github.com/zricethezav/gitleaks/blob/master/config/gitleaks.toml) that it uses to identify data it strongly suspects are secrets. It will catch secrets that follow the format of common secrets like AWS secret keys and GCP API keys, or even obscure secrets like "Etsy Access Tokens." Is there something that we want to add? (no action)
