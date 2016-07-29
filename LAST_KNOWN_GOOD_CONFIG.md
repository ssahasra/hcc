Overview
========
This document shows the last known good configuration for the following
projects:
- ToT HCC
- ToT HCC Clang
- upstream Clang (where ToT HCC Clang is synchronized with)
- upstream LLVM
- upstream LLD

ToT HCC
-------
- clone: git@github.com:RadeonOpenCompute/hcc.git
- branch: clang_tot_upgrade
- commit: aa2ad873773061d399b4bfbe3051f0ea948ba6c9

ToT HCC Clang
-------------
- clone: git@github.com:RadeonOpenCompute/hcc-clang-upgrade.git
- branch: clang_tot_upgrade
- commit: 7cb25e2394a1975d34cd102cc9fe1d969de9e604

upstream Clang
--------------
- clone: git@github.com:RadeonOpenCompute/hcc-clang-upgrade.git
- branch: upstream
- commit: 5d0d64dcfd3f84fc684c40a79f88fa223b3852b6

upstream LLVM
-------------
- clone: https://github.com/llvm-mirror/llvm
- branch: master
- commit: c6c1814d38905ae31faf406e74a557516399bcda

upstream LLD
------------
- clone: https://github.com/llvm-mirror/lld 
- branch: master
- commit: acddb44526eb60458db830a4ef1ff25a6c0f72db

How to synchronize ToT HCC with upstream
========================================
This section shows the step to synchronize ToT HCC with upstream projects, such
as Clang, LLVM, and LLD. The process is currently carried out manually but it
could and should be automated.

Upstream Clang, LLVM, and LLD all sit in different git repositories and they
are almost changed daily. Sometimes a change upstream may affect several
projects, but it's usually not easy to figure it out from the commit log.

Generally speaking, the process goes like this:

Process A: merge upstream Clang

1. Add git remote for upstream Clang
2. Fetch upstream Clang commits
3. Merge upstream Clang with ToT HCC Clang
4. Build merged ToT HCC Clang
   - For failures introduced by changes in LLVM / LLD, execute Process B
5. Quick sanity tests on merged ToT HCC Clang
6. Update LAST_KNOWN_GOOD_CONFIG.md (this document)
7. Push everything

Process B: merge upstream LLVM / LLD

1. Fetch upstream LLVM commits
2. Fetch upstream LLD commits
3. Build upstream LLVM / LLD
4. Update LAST_KNOWN_GOOD_CONFIG.md (this document)
5. Remove ToT HCC checkout and restart Process A

Detailed step-by-step instructions are in following sections.

Process A: merge upstream Clang
-------------------------------

### Add git remote for upstream Clang
It's assumed there's already a ToT HCC checkout, and also a ToT HCC Clang
checkout, normally found under `compiler/tools/clang` diretory under ToT HCC
checkout.

- Enter `compiler/tools/clang`
- `git remote -v` to check if there's a git remote pointing to:
  `git@github.com:llvm-mirror/clang.git`
- If there's not, add it by:
  `git remote add clang git@github.com:llvm-mirror/clang.git`

### Fetch upstream Clang commits
Assume commands below are carried out in `compiler/tools/clang`.

- `git checkout upstream` : change to the branch to keep upstream commits. The branch contains no HCC-specific codes.
- `git fetch clang` : fetch upstream commits
- `git merge --no-ff clang/master` : get upstream commits merged into upstream branch

### Merge upstream Clang with ToT HCC Clang
Assume commands below are carried out in `compiler/tools/clang`.

- `git checkout clang_tot_upgrade` : change to the main develop branch for ToT HCC Clang
- `git merge upstream` : merge commits from upstream Clang to ToT HCC Clang

Resolve any merge conflicts encountered here.

### Build merged ToT HCC Clang
Assume a ToT HCC build directory is there. If there's not, follow Appendix B to configure one.

- change to ToT HCC build directory
- `make -j16`

Fix any compilation failures if there's any. For failures introduced by changes
in LLVM / LLD. Stop. Execute Process B and restart Process A.

### Quick sanity tests on merged ToT HCC Clang
Assume commands below are carried out in ToT HCC build directory. And ToT HCC
checkout is at `~/hcc_upstream`.

Test with one C++AMP FP math unit test.
```
bin/hcc `bin/clamp-config --build --cxxflags --ldflags` -lm ~/hcc_upstream/tests/Unit/AmpMath/amp_math_cos.cpp
./a.out ; echo $?
```

Test with one grid_launch unit test with AM library usage.
```
bin/hcc `bin/hcc-config --build --cxxflags --ldflags` -lhc_am ~/hcc_upstream/tests/Unit/GridLaunch/glp_const.cpp
./a.out ; echo $?
```

### Update LAST_KNOWN_GOOD_CONFIG.md (this document)

- change to ToT HCC Clang directory
- `git checkout upstream`
- `git rev-parse HEAD` : log the result in "upstream Clang" in the beginning of this document
- `git checkout clang_tot_upgrade`
- `git rev-parse HEAD` : log the result in "ToT HCC Clang" in the beginning of this document
- change to ToT HCC directory
- `git rev-parse HEAD` : log the result in "ToT HCC" in the beginning of this document

### Push everything

- change to ToT HCC directory
- `git add LAST_KNOWN_GOOD_CONFIG.md`
- `git commit`
- `git push`
- change to ToT HCC Clang directory
- `git checkout clang_tot_upgrade`
- `git push`
- `git checkout upstream`
- `git push`

Following steps are to ensure "develop" and "master" branch are kept the same
as "clang_tot_upgrade" branch.
- `git checkout develop`
- `git merge clang_tot_upgrade`
- `git push`
- `git checkout master`
- `git merge clang_tot_upgrade`
- `git push`

Upon reaching here, the merge process is completed.


Process B: merge upstream LLVM / LLD
------------------------------------
Sometimes it's not possible to synchronize with upstream Clang without also
synchronizing with upstream LLVM / LLD. This section explains steps to do that.
Notice by the end of this Process you are asked to *remove* your ToT HCC checkout and restart Process A from scratch. If you have work already applied in ToT HCC Clang it's recommended to stash them elsewhere.

In the near future ToT HCC and ToT HCC Clang should move to a true out-of-source build model to simplify the process.

### Fetch upstream LLVM commits
Assume there is already an upstream LLVM checkout for AMDGPU(Lightning) backend.

- change to upstream LLVM directory
- `git pull`

### Fetch upstream LLD commits
Assume there is already an upstream LLD checkout for AMDGPU(Lightning) backend. Normally it would be in "tools/lld" in LLVM checkout.

- change to upstream LLD directory
- `git pull`

### Build upstream LLVM / LLD
Assume there's a build directory for upstream LLVM / LLD. If there's not, follow Appendix A to configure one.

- change to LLVM build directory
- `make -j16`

### Update LAST_KNOWN_GOOD_CONFIG.md (this document)

- change to upstream LLVM directory
- `git rev-parse HEAD` : log the result in "upstream LLVM" in the beginning of this document
- change to upstream LLD directory
- `git rev-parse HEAD` : log the result in "upstream LLD" in the beginning of this document

### Remove ToT HCC checkout and restart Process A
In ToT HCC there's also an LLVM / LLD checkout which sits in "compiler/" directory, and they would be patched by ToT HCC. It would be complicated to undo the process. So right now the recommended approach is to simply *remove* ToT HCC checkout and restart Process A. You may need to stash your changes in ToT HCC Clang somewhere else before removing ToT HCC checkout.


Appendix A: CMake command for upstream LLVM / LLD
=================================================

```
cmake \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/opt/rocm/llvm \
    -DLLVM_TARGETS_TO_BUILD="AMDGPU;X86" \
    <upstream LLVM checkout directory>
```

Appendix B: CMake command for ToT HCC
=====================================

```
cmake \
    -DCMAKE_BUILD_TYPE=Release \
    -DHSA_LLVM_BIN_DIR=<upstream LLVM build directory>/bin \
    -DHSA_AMDGPU_GPU_TARGET=fiji \
    -DHSA_USE_AMDGPU_BACKEND=ON \
    <ToT HCC checkout directory>
```