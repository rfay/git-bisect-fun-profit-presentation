# **Git Bisect for Fun and Profit**
### with

<img src="images/ddev-logo.svg" alt="DDEV Logo" class="ddev-logo">
https://github.com/rfay/git-bisect-fun-profit-presentation
https://rfay.github.io/git-bisect-fun-profit-presentation/

---

## The Problem

This silly (altered) Drupal 11.3.x is broken — it doesn't install with PHP 8.3

We need to find **which commit broke it**

---

## Git Bisect Basics

```bash
git bisect start
# Check out bad commit
git bisect bad   # mark current commit as broken
# Check out good commit
git bisect good  # mark a known-good commit
# git checks out the midpoint
git bisect good  # or bad, based on what you find
# repeat until git identifies the culprit
git bisect reset # return to HEAD when done
```

Binary search through your commit history.

---

## Setup

```bash
git clone https://github.com/rfay/git-bisect-example
ddev config --php-version=8.3
ddev composer install
ddev launch  
# fails with "Your PHP installation is too old"
```

PHP 8.4 works. PHP 8.3 doesn't. Drupal 11.2.0 was fine.

---

## Start Bisecting

```bash
git bisect start
git bisect bad          # current HEAD is broken
git checkout 11.2.0
ddev launch             # verify it works
git bisect good         # mark it good
```

Git will now check out the midpoint between good and bad

---

## The Manual Process

At each step, check if the install works:

```bash
ddev launch 
# does /core/install.php show "Choose language"?
```

Then tell git what you found:

```bash
git bisect good   # or
git bisect bad
```

Repeat until git identifies the bad commit

---

## Automate It

```bash
curl -s \
  https://git-bisect-example.ddev.site/core/install.php \
  | grep -q "<title>Choose language"
```

Returns `0` (success) when working

---

## The Automation Script

Store in `~/tmp/check-installable.sh` (outside the repo!)

```bash
#!/bin/bash
sleep 1
ddev mutagen sync >/dev/null 2>&1
sleep 1
ddev composer install --no-interaction >/dev/null 2>&1
curl -sfL https://git-bisect-example.ddev.site/core/install.php \
  | grep "<title>Choose language" >/dev/null
exit $?
```

---

## Run the Automated Bisect

```bash
git bisect reset
git bisect start 11.3.x 11.2.0   # <bad> <good>
git bisect run ~/tmp/check-installable.sh
```

Git runs the script automatically at each step.

---

## Caveats

Real debugging requires some extra work, like maybe `composer install` or fix database, etc. Those can be factored into the script. Here:

* Each bisect step checks out fresh code — rebuild what's needed (e.g. `composer install`)
* Some situations require a fresh database or `drush si` at each step
* On macOS/Windows, Mutagen needs a chance to sync large git checkouts — add `ddev mutagen sync` to your script

---

## Resources

* Docs: https://git-scm.com/docs/git-bisect
* One of many tutorials: https://dev.to/alvesjessica/using-git-bisect-to-find-the-faulty-commit-25gf


* Git Bisect Example: https://github.com/rfay/git-bisect-example
* Presentation: https://rfay.github.io/git-bisect-fun-profit-presentation/
* Presentation repository: https://github.com/rfay/git-bisect-fun-profit-presentation

