# Jekyll Blog

If you run Olympum `gcloud` commands from this repo, use the isolated Cloud SDK
setup documented in `../mithran-infra/docs/developer-setup.md`. Avoid
committed repo-local env templates and avoid relying on
`CLOUDSDK_CORE_ACCOUNT=...` prefixes as the steady-state workflow.

```
$ bundle update # update gems (optional)
$ bundle exec jekyll serve --watch
```
