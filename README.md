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


## Fuzzing chromium with AFL++ instead of AFL

Modify pdfium/third_party/afl/BUILD.gn, copy the compiled AFLplusplus folder to src, rename afl-cc to afl-clang and afl-c++ to afl-clang++

```
--- BUILD.gn.bak	2022-03-11 09:57:06.393190381 +0800
+++ BUILD.gn	2022-03-09 10:57:43.640532334 +0800
@@ -2,13 +2,10 @@
 # Use of this source code is governed by a BSD-style license that can be
 # found in the LICENSE file.
 
+# Modified by Hanyu to fit AFLplusplus.
+
 group("afl") {
   deps = [
-    ":afl-cmin",
-    ":afl-fuzz",
-    ":afl-showmap",
-    ":afl-tmin",
-    ":afl_docs",
     ":afl_runtime",
   ]
 }
@@ -32,19 +29,11 @@ source_set("afl_runtime") {
     "//build/config/gcc:symbol_visibility_hidden",
   ]
 
-  sources = [
-    "src/llvm_mode/afl-llvm-rt.o.c",
-  ]
+  sources = [ "src/afl-llvm-rt-lto.o",
+              "src/afl-llvm-rt-64.o",
+            ]
 }
 
-afl_headers = [
-  "src/alloc-inl.h",
-  "src/config.h",
-  "src/debug.h",
-  "src/types.h",
-  "src/hash.h",
-]
-
 config("afl-tool") {
   cflags = [
     # Include flags from afl's Makefile.
@@ -63,60 +52,4 @@ config("afl-tool") {
     # we do not use. Therefore its value is unimportant.
     "-DBIN_PATH=\"$root_build_dir\"",
   ]
-}
-
-copy("afl-cmin") {
-  # afl-cmin is a bash script used to minimize the corpus, therefore we can just
-  # copy it over.
-  sources = [
-    "src/afl-cmin",
-  ]
-  outputs = [
-    "$root_build_dir/{{source_file_part}}",
-  ]
-  deps = [
-    ":afl-showmap",
-  ]
-}
-
-copy("afl_docs") {
-  # Copy the docs folder. This is so that we can use a real value for for
-  # -DDOC_PATH when compiling.
-  sources = [
-    "src/docs",
-  ]
-  outputs = [
-    "$root_build_dir/afl/{{source_file_part}}",
-  ]
-}
-
-executable("afl-fuzz") {
-  # Used to fuzz programs.
-  configs -= [ "//build/config/sanitizers:default_sanitizer_flags" ]
-  configs += [ ":afl-tool" ]
-
-  sources = [
-    "src/afl-fuzz.c",
-  ]
-  sources += afl_headers
-}
-
-executable("afl-tmin") {
-  configs -= [ "//build/config/sanitizers:default_sanitizer_flags" ]
-  configs += [ ":afl-tool" ]
-
-  sources = [
-    "src/afl-tmin.c",
-  ]
-  sources += afl_headers
-}
-
-executable("afl-showmap") {
-  configs -= [ "//build/config/sanitizers:default_sanitizer_flags" ]
-  configs += [ ":afl-tool" ]
-
-  sources = [
-    "src/afl-showmap.c",
-  ]
-  sources += afl_headers
-}
+}
```
