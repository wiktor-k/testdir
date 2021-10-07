# Semi-persistent, scoped test directories

This crate aims to make it easier to use temporary directories in
tests, primarily by calling the `testdir!()` macro somewhere in a test
function.  The direcetories are structured per-scope and per-test and
will be available for inspection after the test has finished.  On
subsequent test runs older generations of test directories will be
cleaned up.

If you've ever used [pytest's `tmp_path` or `tmpdir`
fixtures](https://docs.pytest.org/en/stable/reference.html#tmp-path)
this pattern should be similar.

By default test directories are created in cargo's target directory:
```
target/testdir-$N/module/path/test_name
```

Here `$N` is an integer generation number increasing with each `cargo
test` invocation.  Generations older than the 8 most recent ones are
removed on the next `cargo test` run.

There is also a symlink pointing to the most recent generation:
```
target/testdir-current -> testdir-$N`
```

## Example

Even when executing this with `cargo test --jobs=1` these tests will
pass as each gets their own unique directory:
```no_run
// E.g. in lib.rs

mod tests {
    use std::path::PathBuf;
    use testdir::testdir;

    #[test]
    fn test_write() {
        let dir: PathBuf = testdir!();
        let path = dir.join("hello.txt");
        std::fs::write(&path, "hi there").ok();
        assert!(path.exists());
    }
    
    #[test]
    fn test_read() {
        let dir: PathBuf = testdir!();
        let path = dir.join("hello.txt");
        assert!(!path.exists());
    }
}
```

Afterwards you can inspect the directories and they should look
something like this:
```sh
$ tree target/
target/
+- testdir-0
|    +- cratename
|         +- tests
|              +- test_read
|              +- test_write
|                   +- hello.txt
+- testdir-current -> testdir-0
```

## Feedback and contributing

The code lives in a mercurial repository at
https://hg.sr.ht/~/flub/testdir from which you can clone the
repository.

For any feedback, bug reports or patches please send an email to
[~flub/testdir@lists.sr.ht](mailto:~flub/testdir@lists.sr.ht) (or use
[u.flub.testdir@lists.sr.ht](mailto:u.flub.testdir@lists.sr.ht) if
your mail client does not accept the former).

The mailing list archive can be browsed at
[https://lists.sr.ht/~flub/testdir](https://lists.sr.ht/~flub/testdir).
