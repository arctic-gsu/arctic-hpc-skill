# Maintaining arctic-hpc

The skill summarizes the ARCTIC wiki. Keep it in step with the wiki and the cluster.

## What may go in

- Only information that's public on the ARCTIC wiki, or that any user can see on the cluster (for example, with `sinfo`, `module avail`, or a test job).
- Never add staff-only partitions or nodes, group or account names, staff usernames, internal hostnames or IP addresses, admin procedures, or unpublished hardware, pricing or allocation details.
- No claims without a source. Each section of `references/arctic-reference.md` ends with a `Source:` line listing its wiki pages; add the page when you add a fact.

## When to update

- After a wiki page listed in a `Source:` line changes: re-read that section against the page.
- After cluster changes: new or retired nodes, GPUs, partitions or modules; storage quotas; login hosts; the LLM service's models.
- After GSU's AI Tool Review Registry changes (section 8 and SKILL.md's AI policy section).
- At least once a semester: compare section 1, 5 and 6 tables with `sinfo` and `module avail`.

## How to release

1. Edit `SKILL.md` and `references/arctic-reference.md`. Keep the `SKILL.md` description at 200 characters or fewer and the frontmatter to `name`, `description` and `metadata` (some tools reject other fields).
2. Bump `metadata.version` and `metadata.updated` in `SKILL.md`, and update "Last verified" at the top of the reference.
3. Add an entry to `CHANGELOG.md`.
4. Commit, then tag and push: `git tag v1.1.0 && git push origin main v1.1.0`. The release workflow builds `arctic-hpc.zip` and attaches it to the release; the README's download link always points to the latest one.
5. OpenCode users get the update with `git pull` in their skill folder.
