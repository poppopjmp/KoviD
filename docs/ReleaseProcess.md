# Release Process Steps

This document outlines the steps required to prepare, validate, and publish a new release.

## Prerequisites
- Ensure the master branch is up-to-date and all changes intended for the release are merged.
- Confirm that the project builds successfully.
- Confirm that CI/CD passes.
- All tests, both native and cross-compiled, must pass.

## Steps

### 1. Version Bump
1. Determine the next release version according to the versioning policy.
2. Update the version number.

### 2. Update Changelog
1. Open `CHANGELOG.md`.
2. Add a summary of changes, including new features, bug fixes, and any known issues since the last release.
3. Ensure notes are clear, accurate, and helpful.

### 3. Run Native Tests
From the project root directory:
```bash
cd KoviD && make PROCNAME="myprocname" TEST_ENV=1
mkdir build && cd build
cmake ../ -DCMAKE_C_COMPILER=gcc && make PROCNAME="myprocname" TEST_ENV=1
# Run tests with a single thread:
make check-kovid -j1
```

### 4. Run Cross-Compilation Tests

Deploy configuration:
```bash
mkdir build_cross_deploy && cd build_cross_deploy
cmake ../ -DKOVID_LINUX_VERSION=5.10 \
          -DKERNEL_DIR=projects/private/kovid/linux \
          -DCROSS_TESTS=ON \
          -DCMAKE_C_COMPILER=gcc \
          -DDEPLOY=1 \
          -DNATIVE_TESTS=OFF
make PROCNAME="myprocname" DEPLOY=1 TEST_ENV=1
make check-kovid
```

Debug configuration:
```bash
mkdir build_cross_debug && cd build_cross_debug
cmake ../ -DKOVID_LINUX_VERSION=5.10 \
          -DKERNEL_DIR=projects/private/kovid/linux \
          -DCROSS_TESTS=ON \
          -DCMAKE_C_COMPILER=gcc \
          -DDEPLOY=0 \
          -DNATIVE_TESTS=OFF
make PROCNAME="myprocname" DEPLOY=0 TEST_ENV=1
make check-kovid
```

### 5. Configure GitHub Actions Workflow

1. Open `.github/workflows/build.yml`.
2. Ensure the workflow includes a matrix strategy to build different versions of the software.
3. Verify that the workflow is triggered on push and pull request events for the `master` branch.
4. Confirm that the workflow includes steps to check out the repository, install dependencies, build the software, and run tests for each version specified in the matrix.

### 6. Finalize Artifacts

1. Confirm all binaries, documentation, and test results are ready.
2. Clean up any temporary files or directories not required in the release.

### 7. Tag the Release

From the repository:
```bash
git tag -a vX.Y.Z -m "Release vX.Y.Z"
git push origin vX.Y.Z
```
Replace vX.Y.Z with your new version.

### 8. Publish the Release

1. Create a new release on GitHub.
2. Use the newly created tag.
3. Include CHANGELOG.md content in the release description.
4. Optionally attach binary.

### 9. Announcement (Optional)

Notify users and team members about the release, highlighting major changes and improvements.
