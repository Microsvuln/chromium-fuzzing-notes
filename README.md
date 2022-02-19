# chromium-fuzzing-notes

## Chromium windows compilation 

```
is_debug = false
is_asan = true
enable_nacl = false
symbol_level = 1 
blink_symbol_level = 0
use_sanitizer_coverage = true
sanitizer_coverage_flags = "trace-pc-guard,bb"
```

Note : Never compile with `is_component_build = true` when compiling with `use_sanitizer_coverage = true`, this will make a conflict, get rid of this on windows sanitizer coverage build mode .
