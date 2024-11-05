# sqlite-cmake

CMake build for SQLite amalgamation. It's basically all C and header files concatenated into one .c and .h files. The src/ folder contains this amalgamation as is not intended to be manually edited. Just download latest version and overwrite the files when you wish to update.

# Building

To build with command line tools included:
```
mkdir target
cd target
cmake -DBUILD_CLI=ON .. && make
```

# Links

* SQLite home page (see for docs, new releases etc.) https://www.sqlite.org/
