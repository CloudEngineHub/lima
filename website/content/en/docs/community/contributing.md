---
title: Contributing
weight: 20
---

## Reporting issues

Bugs and feature requests can be submitted via <https://github.com/lima-vm/lima/issues>.

For asking questions, use [GitHub Discussions](https://github.com/lima-vm/lima/discussions) or [Slack (`#lima`)](https://slack.cncf.io).

For reporting vulnerabilities, see <https://github.com/lima-vm/.github/blob/main/SECURITY.md>.

## Contributing code

### Developer Certificate of Origin

Every commit must be signed off with the `Signed-off-by: REAL NAME <email@example.com>` line.

Use the `git commit -s` command to add the Signed-off-by line.

See also <https://github.com/cncf/foundation/blob/main/policies-guidance/dco-guidelines.md>.

### Licensing

Lima is licensed under the terms of [Apache License, Version 2.0](https://github.com/lima-vm/lima/blob/master/LICENSE).

See also <https://github.com/cncf/foundation/blob/main/policies-guidance/allowed-third-party-license-policy.md> for third-party dependencies.

### Sending pull requests

Pull requests can be submitted to <https://github.com/lima-vm/lima/pulls>.

It is highly suggested to add [tests](../../dev/testing/) for every non-trivial pull requests.
A test can be implemented as a unit test rather than an integration test when it is possible,
to avoid slowing the integration test CI.

### Merging pull requests

[Committers](../governance) can merge pull requests.
[Reviewers](../governance) can approve, but cannot merge, pull requests.

A Committer shouldn't merge their own pull requests without approval by at least one other Maintainer (Committer or Reviewer).

This rule does not apply to trivial pull requests such as fixing typos, CI failures,
and updating image references in templates (e.g., <https://github.com/lima-vm/lima/pull/2318>).
