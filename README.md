# Google Docs

Create, import, inspect, edit, and deliberately share native Google Docs through
the [Google Workspace CLI](https://github.com/googleworkspace/cli). Install
`google-docs@package-manager` in Codex or Claude, make `gws` available on PATH,
and use your own existing Google Workspace authentication.

The skill checks the live API schema, resolves the exact document, protects
concurrent edits, and verifies the saved result. Optional document-generation
tools can produce a rendered DOCX before import when layout matters. The Docs
API remains available for ordinary native document edits. Document content,
account credentials, and collaborator access stay outside the package.

The package has no runtime Node dependencies. Run `npm test` for structural
validation; it does not access a Google account. See `PROVENANCE.md`,
`ICON-SOURCES.md`, and `LICENSE`.
