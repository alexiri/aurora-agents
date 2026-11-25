---
name: change-check
use-it-when: "Use before every push: it checks your change against the size limits, the forbidden-paths and secrets rules, and tells you what to fix."
---
`change-check` reads the diff of a repository and holds it against the rules in your instructions. It sends nothing anywhere.

```sh
C="$SKILLS_DIR/change-check/scripts/change-check"
cd <the repository>
"$C" --base origin/<target branch> [--forbid <path or folder>]...
```

Pass `--forbid` once for each path the repos memory lists under "never touch", and once for **every file the repository's `.gitlab-ci.yml` includes** (read the file to find them; don't change it). `--allow <path>` exempts one default path, only when the repos memory says it's a normal place for a fix in that repository. It works from the repository's top folder and looks at everything that differs from the base: commits on your branch, the index, unstaged changes and new files. It checks:

- **Size:** at most 300 changed lines (added plus deleted) and 10 files (`--max-lines` and `--max-files` change the limits only when the repos memory says so).
- **Forbidden paths:** CI configuration (`.gitlab-ci.yml`, `.gitlab-ci/`, `.gitlab/`, `.github/`, `Jenkinsfile`, `azure-pipelines.yml`, `.circleci/`, `.travis.yml`) and everything the tests or the CI execute: `Makefile`/`makefile` and `*.mk`, `Rakefile`, `Gemfile*`, `Puppetfile`, `.fixtures.yml`, `.rspec`, `spec_helper*`, `conftest.py`, `tox.ini`, `noxfile.py`, `package.json`, `pyproject.toml`, `setup.py`, `setup.cfg`, `requirements*.txt`, `.pre-commit-config.yaml`, `.gitmodules`; plus every `--forbid` path. The code under test is fine to change; the files that run the tests are not. Also pass `--forbid` for every repository file a CI job's `script:` runs. If the repos memory says one of the default paths is a normal place for a fix in that repository, pass `--allow <path>`; otherwise a change there needs a person.
- **Symlinks, submodules and file-mode changes:** refused.
- **Secrets:** added lines that look like credentials (`token`, `password`, `secret`, `api_key`, `credential` names with a value, passwords in URLs, private key headers, `glpat-…`, `AKIA…`, bearer tokens), and files named like secrets (`.env`, `*.pem`, `*.key`, `id_rsa*`).
- **Binary files, very long lines:** refused: you change text, by hand.
- **Deleted files:** more than 3 is refused. Deleting is rarely what a ticket means.
- **Nothing changed:** an empty diff is a problem (declining is a result; an empty merge request isn't).

Exit status 0 means it's fine; 1 means problems, each printed on stderr with its path; 2 is bad usage. Fix what it reports. If a forbidden path really is the fix, the ticket needs a person: decline.

A pass doesn't mean the change is right. It means the change is small and clean. Reading the diff is still your job.
