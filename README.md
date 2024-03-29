# action-dart-compile

This action compiles Dart bins and uploads them to the GitHub release.

## Not use this action

You just add next action yml file.

```yml
name: Add binaries to release
run-name: Add binaries to release ${{ github.event.release.tag_name }}

on:
  release:
    types:
      - created

env:
  GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}

permissions:
  contents: write

jobs:
  upload:
    strategy:
      matrix:
        os: [windows, macos, ubuntu]
    runs-on: ${{ matrix.os }}-latest
    name: Upload ${{ matrix.os }} binaries to release ${{ github.event.release.tag_name }}
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      - uses: dart-lang/setup-dart@v1
        with:
          sdk: stable
      - name: compile and upload
        shell: bash
        run: |
          dart pub get
          dart analyze
          export DART_BINS=$(ls bin/*.dart)
          for dart_file in $DART_BINS; do
              echo "Compiling $dart_file"
              dart compile exe $dart_file
          done
          chmod +x bin/*.exe
          export TARGET_GZ=${{ matrix.os }}_${{ github.event.release.tag_name }}.tar.gz
          tar -czvf $TARGET_GZ bin/*.exe
          gh release upload ${{ github.event.release.tag_name }} $TARGET_GZ
```

## Use the action

```yaml
name: Add binaries to release
run-name: Add binaries to release ${{ github.event.release.tag_name }}
on:
  release:
    types:
      - created

jobs:
  upload:
    strategy:
      matrix:
        os: [windows, macos, ubuntu]
    runs-on: ${{ matrix.os }}-latest
    name: Upload ${{ matrix.os }} binaries to release ${{ github.event.release.tag_name }}
    steps:
      - name: Upload binaries to release ${{ github.event.release.tag_name }} on ${{ matrix.os }}
        uses: caijinglong/action-dart-compile@v1
    env:
      GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    permissions:
      contents: write
    
```
