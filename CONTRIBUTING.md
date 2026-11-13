# Contributing to westailabs.com

1. Branch off `develop`: `git switch develop && git switch -c feat/<short-name>` (or `fix/`, `docs/`, `chore/`).
2. Keep changes small and focused. Use conventional commit messages (`feat:`, `fix:`, `docs:`, `chore:`).
3. Run the checks before you merge:

   ```bash
   JEKYLL_ENV=production bundle exec jekyll build
   ```
4. Merge back into `develop` with `git merge --no-ff` (or `git merge --squash` for a single clean commit), then delete the branch.
5. `main` is the release branch. Do not commit to it directly.

Agent and tool instructions live in [AGENTS.md](AGENTS.md).
