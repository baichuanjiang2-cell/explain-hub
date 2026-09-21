# Security policy

## Scope

explain-hub is an agent skill: markdown rules, HTML templates, and a
vendored copy of diagram-design. It ships no executable code of its own.
Security issues that matter here are:

- Prompts or templates that instruct an agent to do something unsafe
  (exfiltrate data, damage the workspace, mislead the user about what was
  verified).
- The cleanup rules (`_repos/` temp clones, temp JSON) accidentally
  deleting user data, or leaving secrets behind.
- Anything in the vendored engine that turns generated HTML into an
  attack surface (e.g. scripts embedded in diagram output).

## What is not a vulnerability

- Rate limiting, clone failures, or Chromium download failures — those are
  handled by the documented fallback chains.
- The skill fetching **public** GitHub data (stars, READMEs, metadata) over
  HTTPS. It does not collect or transmit your credentials, tokens or stars
  anywhere beyond the endpoints an agent would call anyway.

## Reporting

Please use
[GitHub private vulnerability reporting](https://docs.github.com/en/code-security/security-advisories/guidance-on-reporting-and-reviewing-security-vulnerabilities/privately-reporting-a-security-vulnerability)
on this repository, or open an issue marked **security** if reporting
privately is unavailable. Include the file and section at fault and, if
applicable, a minimal scenario where the agent follows the rule into the
unsafe behavior.
