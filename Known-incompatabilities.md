**paper_trail** - Sorcery doesn't play nice with paper_trail due to a before_filter that paper_trail adds behind the scenes that calls 'current_user' a bit too early for sorcery. This issue *might* be solved by calling
```
    prepend_before_filter :require_login
```

instead of the usual
```
    before_filter :require_login
```

Though this hasn't been proven (please report success).