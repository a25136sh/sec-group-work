# Build

```
docker build -t sec-group-work .
```

# Run

```
docker run -it --rm sec-group-work
```

このままだと終了時にデータ消えるのでご注意ください。

# Verify installation
```
klee --version
afl-clang-fast -v

cppcheck --version
flawfinder --version
rats --help
```
