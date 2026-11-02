# First patched Dependabot run

Run these commands in your Ubuntu VM, with Docker and Git installed and working. This first run uses Dependabot's development container and dry-run script. It runs a real update check and prints the proposed changes; it does not create a PR.

```bash
git clone https://github.com/dependabot/dependabot-core.git
cd dependabot-core
git checkout v0.399.0
git switch -c my-logging-patch
```

Add the logging line to Dependabot's common file fetcher:

```bash
python3 - <<'PY'
from pathlib import Path
path = Path("common/lib/dependabot/file_fetchers/base.rb")
text = path.read_text()
needle = "      def initialize(source:, credentials:, repo_contents_path: nil, options: {}, update_config: nil)\n"
assert text.count(needle) == 1, "Expected initializer not found exactly once"
path.write_text(text.replace(needle, needle + '        puts "$$$$$$ hello from patch $$$$$$"\n'))
PY
git diff
```

Start the Go Modules development container:

```bash
bin/docker-dev-shell go_modules
```

Inside the container, run:

```bash
bin/dry-run.rb go_modules rsc/quote
```

Look for `$$$$$$ hello from patch $$$$$$` in the output. That line comes from the modified Dependabot code when it initializes the file fetcher. Type `exit` to leave the container.

If GitHub API authentication is needed, create a GitHub token with only the access needed for the repository and pass it to the development shell as documented in the upstream development guide; do not commit the token.

This is a manual first step. Running automatically, creating PRs, and building a patched updater image for Dependabot CLI are separate steps.
