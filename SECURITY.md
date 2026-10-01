# Security and privacy policy

This repository contains prose, not software. Its security surface is therefore a single
question: does anything here disclose something it should not?

## Reporting

Please report privately through GitHub's "Report a vulnerability" button on the repository's
Security tab. Do not open a public issue, because a public issue about a disclosure repeats
the disclosure.

You will get an acknowledgement within 72 hours, and anything confirmed is removed the same
day.

## What must not be here

These case studies describe systems built for a law firm and its clients. Every one of them
handles personal data, and one of them handles the debt records of several hundred named
individuals. **None of that data, and no identifier that could be used to find it, belongs in
this repository.**

Specifically, a finding would be any of:

- A real person's name, CPF, RG, address, telephone number or e-mail address.
- A real company's name, CNPJ, or a certificate serial number.
- A Brazilian court case number (CNJ) that passes its own check digits, meaning it could be a
  real case.
- A vehicle chassis number, licence plate or RENAVAM.
- The name of a client, a counterparty, or an employee.
- A credential of any kind: a token, an app or tenant id, a connection string, an internal
  hostname or an S3 bucket name.
- A screenshot or a log excerpt containing any of the above.

Every number that appears in these write-ups is a count, a duration or a line total. Where a
system's data shape matters, it is described by its column names and never by its contents.

## Why there is no code here

Where a system could be rewritten into something generic, it was, and it was published as its
own repository with synthetic data — those are linked from the README. What is left here is
the part that could not be: either because the code belongs to somebody else, or because the
system is inseparable from the records it processes.

A case study was the honest option. Publishing a sanitised copy of code that was written
under contract would not have been.

## Supported versions

Only the `main` branch. Corrections are made in place.
