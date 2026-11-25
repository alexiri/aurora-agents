---
name: draft-merge-request
use-it-when: "Use when your change is made and checked and you are about to branch, commit, push and open the draft merge request: names, messages, the description, and what you must never do."
---
A draft merge request is how a change reaches a person. Make it easy to read and impossible to merge by accident.

## Branch and commit

- Branch: `fixer/<ticket-key-in-lowercase>-<short-slug>`, for example `fixer/mir-1506-limit-snapshot-clients`; for a change described in a conversation, `fixer/chat-<yyyymmdd>-<short-slug>`. Create it from the target branch named in `repos-<service>` after a fresh fetch. Never commit to the target branch or to any branch you didn't create. Before you create one, look for an earlier attempt: `list_branches` with `search=fixer/<key>-` (the slug can differ between runs). If one exists, check it's yours (every commit on it authored by your GitLab account; `ticket-handoff` says how). If it is, carry on from it (`ticket-handoff` says when); if it isn't, don't touch it and report it.
- One commit if you can. Message in plain English, no voice: `<TICKET-KEY>: <what it does, imperative, under 72 characters>` (no key prefix for a change from a conversation), then a blank line and a few lines on why. Follow the repository's own commit style if it has one (check `git log`, `AGENTS.md`).
- Commit as whoever git is already set up as. If `git config user.email` is empty, commit with `git -c user.name='The Fixer' -c user.email=<the e-mail from get_current_user> commit …`; don't edit git config, and never sign as a person.
- Never include files you didn't mean to: look at `git status` and `git diff --cached --stat` before committing.

## Push

`git push origin <branch>` over HTTPS through the gitlab app (never push with a token in the URL; the credentials are added for you). Never `--force`, never push tags, never push to anything but your own branch. A push may wait for approval: if it does, report it as pending.

## The merge request

Open it with `create_merge_request` (project by numeric id; JSON body so the approval can show it):

- `title`: `Draft: <TICKET-KEY>: <summary>` (`Draft: <summary>` for a change from a conversation). The `Draft: ` prefix is what makes it unmergeable; always include it.
- `source_branch`, `target_branch`: yours, and the one the repos memory names.
- `remove_source_branch: true`, `squash: true` unless the repos memory says otherwise.
- `labels` (a comma-separated string) and `reviewer_ids`: only what the repos memory lists. Don't invent any.
- `description`, in this shape:

The first line is the effect, in a sentence, if the repos memory asks for that ("what will change on the machines once it's deployed"); otherwise `Fixes <TICKET-KEY>.` The description:

```markdown
<The effect line, as above.> Ticket: <TICKET-KEY> (<link to the ticket>).

## What this changes
<The files and the idea, in a few lines.>

## Why
<How it answers the problem. If the ticket's cause was a hypothesis, say that it still is, and with what confidence.>

## Tests
<The commands you ran and what they printed (the last few lines), or "not run" with the reason.>

## To check it worked, after deploy
<The ticket's verification query, verbatim, and the expected result.>

## Left alone on purpose
<Anything related you did not touch, any doubt you have.>

---
Made by The Fixer from <TICKET-KEY>. Draft until a person says otherwise.
```

(The closing line is where the voice may show, and only if tests ran. If they didn't, keep the line above and add, after it: "Tests didn't run, so look closer than usual.")

Keep the voice to the closing line. The first line and the sections are plain: they're for engineers in a hurry.

## After it's open

- Read the merge request back with `get_merge_request` and check: it is a draft, the branch and target are right, the diff (`get_merge_request_diffs`) is what you meant and nothing else, and the pipeline (if one started) isn't obviously broken by you. If it is, say so in a comment on the merge request (`create_merge_request_note`); don't hide a red pipeline.
- Never mark it ready, never merge, never close someone else's, never delete a branch.
- Link it from the ticket (`ticket-handoff`).
