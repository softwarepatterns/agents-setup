---
name: readme-ownership

description: Create, maintain, review documentation ownership boundaries for readme files and file-level comments.

---


# Readme Ownership

Documentation MUST follow ownership boundaries. README.md files only describe what is within its scope. File-level comments only describe that file.

For the purposes of this skill, README file ownership boundaries include whatever language-specific equivalent is to a module, package, dll, crate, application, server, etc.

IMPORTANT: These rules are NOT required to be explicit headers, simply required prose.

## README.md files

A README.md MUST exist for every ownership boundary, except where the file would be read as product content or produced by the build. For example, every TypeScript/Rust/Python module MUST have a README file. Other situations MAY have README files, but those READMEs do not fall under these guidelines — they are product content or build output, not internal documentation, e.g. installed agent skill directories, public web folders, ./dist, ./target, generated static content.

A README file MUST include in its prose:
- **Ownership Boundary** - such as `this module`, `this package`, `this application`, `this project`
- **Purpose** - what justifies its existence
- **Design** - architectural approach, including particular algorithms, constraints, intended usage
- **Requirements** - invariants, rules, and decisions shared across multiple files
- **Examples** - demonstrated usage of the contents

README files NEVER describe or link to files above or outside their own directory, but may link to remote repositories or documentation. Rules and invariants MUST be placed in the lowest, nearest README whose directory contains its full scope.



## File-level comments

The file-level comments MUST include:
- **Purpose** - what justifies its existence,
- **Design** - architectural approach, including particular algorithms, constraints, intended usage
- **Requirements** - invariants, rules, and decisions
- **Examples** - demonstrated usage of the file contents

File level comments are required for any code file where comments are legal, idiomatic, and free, and NEVER where the format forbids them or where they ship to an end user as payload with no consumer. Therefore Rust, TypeScript, Python, C++, C#, and similar languages MUST include file-level comments, while languages like HTML and JSON NEVER include file-level comments.

File comments NEVER describe or link to other files, but may link to remote repositories or documentation.

## Agent instructions

Ensure `AGENTS.md` requires agents to meet or exceed these documentation standards whenever making affected changes, or propose to the user adding the following requirements if missing:

"When editing a README.md file or a file-level comment, agents must maintain standards equal to or exceeding requirements in the `readme-ownership` skill. Reviewers are required to verify any changes meet or exceed the skill requirements, and that paragraphs and examples are coherent and consistent with the rest of the file."
