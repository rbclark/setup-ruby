# use-ruby

This action downloads a prebuilt ruby and adds it to the `PATH`.

It is very efficient and takes about 5 seconds to download, extract and add the given Ruby to the `PATH`.
No extra packages need to be installed.

### Supported Versions

This action currently supports these versions of MRI, JRuby and TruffleRuby:

| Interpreter | Versions |
| ----------- | -------- |
| Ruby | 2.3.0 - 2.3.8, 2.4.0 - 2.4.9, 2.5.0 - 2.5.7, 2.6.0 - 2.6.5, 2.7.0 |
| JRuby | 9.2.9.0 |
| TruffleRuby  | 19.3.0, 19.3.1 |

The recommended way to refer to these versions is:
`2.3`, `2.4`, `2.5`, `2.6`, `2.7`, `jruby`, `truffleruby`.  
That way, the latest compatible release is used, which contains the most bug fixes.

See https://github.com/eregon/ruby-install-builder/blob/metadata/versions.json
for the always up-to-date list of available Ruby versions.

Note that Ruby 2.3 and the OpenSSL version it needs (1.0.2) are both end-of-life,
which means Ruby 2.3 is unmaintained and considered insecure.
On Windows, Ruby 2.4 uses OpenSSL 1.0.2, which is no longer maintained.

### Supported Platforms

The action works for all GitHub-hosted runners:
`ubuntu-16.04`, `ubuntu-18.04`, `macos-latest` and `windows-latest`.

Ruby 2.3, JRuby and TruffleRuby are not yet supported on `windows-latest`.

The prebuilt rubies are generated by https://github.com/eregon/ruby-install-builder
and on Windows by https://github.com/oneclick/rubyinstaller2.

## Usage

### Single Job

```yaml
name: My workflow
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - uses: eregon/use-ruby-action@v1
      with:
        ruby-version: 2.6
    - run: ruby -v
```

### Matrix

This matrix tests all stable releases of MRI, JRuby and TruffleRuby on Ubuntu and macOS.

```yaml
name: My workflow
on: [push]
jobs:
  test:
    strategy:
      fail-fast: false
      matrix:
        os: [ ubuntu-latest, macos-latest ]
        ruby: [ 2.4, 2.5, 2.6, 2.7, jruby, truffleruby ]
    runs-on: ${{ matrix.os }}
    steps:
    - uses: actions/checkout@v2
    - uses: eregon/use-ruby-action@v1
      with:
        ruby-version: ${{ matrix.ruby }}
    - run: ruby -v
```

### Supported Version Syntax

* engine-version like `ruby-2.6.5` and `truffleruby-19.3.0`
* short version like `2.6`, automatically using the latest release matching that version (`2.6.5`)
* version only like `2.6.5`, assumes MRI for the engine
* engine only like `truffleruby`, uses the latest stable release of that implementation
* `.ruby-version` (also the default value) reads from the project's `.ruby-version` file

### Bundler

Currently, Bundler is guaranteed to be installed for all versions.
If the Ruby ships with Bundler (Ruby >= 2.6), that version is used.
Otherwise (Ruby < 2.6), Bundler 1 is installed when that Ruby was built.

### Caching `bundle install`

You can cache the installed gems with these two steps:

```yaml
    - uses: actions/cache@v1
      with:
        path: vendor/bundle
        key: bundle-use-ruby-${{ matrix.os }}-${{ matrix.ruby }}-${{ hashFiles('**/Gemfile.lock') }}
        restore-keys: |
          bundle-use-ruby-${{ matrix.os }}-${{ matrix.ruby }}-
    - name: Bundle install
      run: |
        bundle config path vendor/bundle
        bundle install --jobs 4 --retry 3
```

When using a single job with a Ruby version, replace `${{ matrix.ruby }}` with the Ruby version.  
When using `.ruby-version`, replace `${{ matrix.ruby }}` with `${{ hashFiles('.ruby-version') }}`.

This uses the [cache action](https://github.com/actions/cache).
The code above is a more complete version of the [Ruby - Gem example](https://github.com/actions/cache/blob/master/examples.md#ruby---gem).
Make sure to include `use-ruby` in the `key` to avoid conflicting with previous caches.

## Limitations

* This action currently only works with GitHub-hosted runners, not private runners.

## Versioning

This action follows semantic versioning with a moving `v1` branch.
This follows the [recommendations](https://github.com/actions/toolkit/blob/master/docs/action-versioning.md) of GitHub Actions.

## Credits

Most of the Windows logic is from https://github.com/MSP-Greg/actions-ruby by MSP-Greg.
