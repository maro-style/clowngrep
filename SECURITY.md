# Security Policy

## Sensitive data

ClownGrep is designed to process material that can contain credentials, PII, customer identifiers and other sensitive data.

Do **not** attach real customer mappings, credentials, leak samples, raw datasets or private output files to public GitHub issues or pull requests.

If you need to demonstrate a bug, create a minimal synthetic dataset using fictitious domains and identities.

## Reporting a vulnerability

If a security issue could expose local data or cause unintended file access, report it privately to the repository maintainer rather than posting proof-of-concept data publicly.

## Operational note

Temporary files are cleaned up during normal execution and handled errors. Abrupt process termination, operating-system crashes or power loss can prevent cleanup, so sensitive environments should include the system temporary directory in their normal data-handling procedures.
