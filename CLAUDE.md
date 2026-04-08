# Project Setup

At the start of every new session, check if the `_bmad` folder exists in the project root. If it does NOT exist, run:

```bash
npx bmad-method install --directory . --modules bmm,cis --tools claude-code --yes
```

This installs the BMad Method tooling required for the project. The `_bmad` folder is not committed to git.
