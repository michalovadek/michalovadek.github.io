# data-external/

Mirrors of data files produced by sibling projects under
`C:\Users\uctqova\Documents\github\<other-repo>\`.

Every file in here MUST have a sibling `<filename>.source.txt` recording:

```
source-repo: <repo-name>
source-path: <path/inside/repo>
source-commit: <git-sha>
pulled-at: YYYY-MM-DD
```

This makes the provenance reproducible without depending on the other repo's
state at a future point in time.

See AGENTS.md §4.
