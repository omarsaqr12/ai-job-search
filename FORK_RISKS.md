# Fork safety notes

This source fork starts from upstream **v1.7.2**, commit
`d7287ec83493102f253282fab07bd88412e7dd50`.
The default branch is `audited-v1.7.2`. Its initial fork changes are documentation only.
It is the preferred baseline among the three snapshots reviewed on 19 October 2026.
It still has the risks below.

## Before using personal information

**This GitHub fork is public. Keep candidate data in a separate local workspace or
private repository.** GitHub forks of this public repository cannot be made private.
Setup writes into tracked files, including `CLAUDE.md`, candidate and behavioral
profiles under `.claude/skills/job-application-assistant/`, and
`cv/main_example.tex`. The `.gitignore` does not protect those tracked files.
Review staged changes before every commit and push. The upstream `SECURITY.md`
overstates how much personal profile content is ignored.

## Remaining risks

| Risk | Consequence and practical control |
| --- | --- |
| Personal data exposure | Publishing a personalized public checkout can expose your CV and profile. Use a separate local or private candidate workspace. Model inference can also send supplied content to your AI provider; local files do not make the workflow offline. |
| Untrusted job pages | Job postings and fetched documents can contain instructions that influence the agent. Restrict file, network, and command access; review external actions and generated applications. Keep optional account integrations disconnected unless needed. |
| Vulnerable optional dependencies | The audited Danish portal dependency graphs include `file-type@16.5.4` and `diff@8.0.2`, with known denial-of-service advisories. Some affected code is bundled inside parent libraries, so a transitive override alone can be insufficient. Keep these portals disabled until their parent packages and bundles are updated and rescanned. |
| Dependency drift and installation | The source has no committed dependency lockfiles, ignores `bun.lock`, and allows version ranges. A future install can resolve different code. Review and retain lockfiles, disable lifecycle scripts during installation, and use frozen installs after reviewing the locks. |
| Job identity and correctness | LinkedIn input handling has an edge case that can select a posting from a URN embedded in an unexpected URL; the fetch still targets LinkedIn. Verify the employer, role, and final posting URL. |
| External service limitations | LinkedIn access can conflict with its terms or trigger rate limits. FreeHire is a third-party service with availability and data-quality limitations. Review each portal's terms and avoid bulk unattended scraping. |
| Codex workflow coverage | Portable portal skills are under `.agents/skills/`. The complete workflow instructions still live under `.claude/`; this fork does not add a complete Codex plugin. Claude permission settings do not configure Codex permissions. |

Public dependency advisories:

- [file-type: GHSA-5v7r-6r5c-r473](https://github.com/advisories/GHSA-5v7r-6r5c-r473), moderate severity.
- [diff: GHSA-73rr-hh4g-fpgx](https://github.com/advisories/GHSA-73rr-hh4g-fpgx), low severity.

The audit confirmed vulnerable dependency code was present. It did not establish
an exploit path from an ordinary job search into those parsers.

## Initial fork settings

- The default branch is based on the exact audited release commit above. The
  inherited `master` branch follows a newer upstream snapshot; use
  `audited-v1.7.2` as the reviewed starting point.
- GitHub Actions were disabled for this fork. Review workflows before enabling
  them; the inherited workflows can install dependencies.
- The four Danish portal skills retain upstream's `enabled: false` setting.
  Setup can enable them when Denmark is selected, so check these flags again
  after onboarding.
- LinkedIn and FreeHire retain `enabled: true`. Their CLIs have zero production
  package dependencies and can run with Bun without installing development types.
- No candidate profile or personal documents were added by the fork operation.

## Audit limits

For the original v1.7.2 snapshot, the audit passed 503 Python cases, 324 Bun test
executions, six TypeScript checks, repository guards, and skill lint. Synthetic
CV and cover-letter PDFs compiled and passed page-count and text checks. A
generic one-result FreeHire search also passed.

Tests used Linux/x64 containers and synthetic inputs. They did not exercise the
complete interactive AI workflow, real applications, OAuth integrations, or
Windows/macOS native execution. Dependency results apply to the recorded audit
resolution, not every future installation. Documentation changes in this fork
do not fix the remaining application risks.
