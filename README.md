# poc-module-name-conflict

How to reproduce random link error:
```bash
# repeat following until link error occurs
docker buildx build --load . -t poc-module-name-conflict:try --progress plain \
    --target builder --no-cache-filter native-builder

# after link error occurs, keep image with tag using --target native-builder
docker buildx build --load . -t poc-module-name-conflict:try --progress plain \
    --target native-builder

# if link error occurs, keep image with tag
docker tag poc-module-name-conflict:try poc-module-name-conflict:link-error
```

How to extract `/poc-module-name-conflict/.build` to `link-error` directory:
```bash
mkdir -p link-error; \
    docker run --rm poc-module-name-conflict:link-error \
    tar czO -C /poc-module-name-conflict .build | tar xzvf - -C link-error
```

Link error output example:
```log
#21 11.81 /root/.swiftpm/swift-sdks/swift-5.10.artifactbundle/aarch64/usr/lib/swift_static/linux/libFoundation.a(CFRegularExpression.c.o):CFRegularExpression.c:function _CFRegularExpressionEnumerateMatchesInString: error: undefined reference to 'uregex_useTransparentBounds_69_swift'
#21 11.81 /root/.swiftpm/swift-sdks/swift-5.10.artifactbundle/aarch64/usr/lib/swift_static/linux/libFoundation.a(CFRegularExpression.c.o):CFRegularExpression.c:function _CFRegularExpressionEnumerateMatchesInString: error: undefined reference to 'uregex_setMatchCallback_69_swift'
#21 11.81 /root/.swiftpm/swift-sdks/swift-5.10.artifactbundle/aarch64/usr/lib/swift_static/linux/libFoundation.a(CFRegularExpression.c.o):CFRegularExpression.c:function _CFRegularExpressionEnumerateMatchesInString: error: undefined reference to 'uregex_setFindProgressCallback_69_swift'
#21 11.81 /root/.swiftpm/swift-sdks/swift-5.10.artifactbundle/aarch64/usr/lib/swift_static/linux/libFoundation.a(CFRegularExpression.c.o):CFRegularExpression.c:function _CFRegularExpressionEnumerateMatchesInString: error: undefined reference to 'uregex_useTransparentBounds_69_swift'
#21 11.81 /root/.swiftpm/swift-sdks/swift-5.10.artifactbundle/aarch64/usr/lib/swift_static/linux/libFoundation.a(CFRegularExpression.c.o):CFRegularExpression.c:function ___CFRegularExpressionDeallocate: error: undefined reference to 'uregex_close_69_swift'
#21 11.81 clang-15: error: linker command failed with exit code 1 (use -v to see invocation)
#21 11.81 error: fatalError
#21 11.81 [15/16] Linking poc-module-name-conflict
#21 DONE 11.9s

#22 [builder 1/1] RUN --mount=type=cache,target=/root/.cache,sharing=locked,id=aarch64-on-aarch64     --mount=type=cache,target=/poc-module-name-conflict/.build,sharing=locked,id=aarch64-on-aarch64     install -v `swift build --configuration release --skip-update --static-swift-stdlib --product poc-module-name-conflict --swift-sdk aarch64-on-aarch64 --show-bin-path`/poc-module-name-conflict /usr/bin
#22 0.304 install: cannot stat '/poc-module-name-conflict/.build/aarch64-unknown-linux-gnu/release/poc-module-name-conflict': No such file or directory
#22 ERROR: process "/bin/sh -c install -v `swift build ${SWIFT_FLAGS} --show-bin-path`/poc-module-name-conflict /usr/bin" did not complete successfully: exit code: 1
------
 > [builder 1/1] RUN --mount=type=cache,target=/root/.cache,sharing=locked,id=aarch64-on-aarch64     --mount=type=cache,target=/poc-module-name-conflict/.build,sharing=locked,id=aarch64-on-aarch64     install -v `swift build --configuration release --skip-update --static-swift-stdlib --product poc-module-name-conflict --swift-sdk aarch64-on-aarch64 --show-bin-path`/poc-module-name-conflict /usr/bin:
0.304 install: cannot stat '/poc-module-name-conflict/.build/aarch64-unknown-linux-gnu/release/poc-module-name-conflict': No such file or directory
------
Dockerfile:202
--------------------
 201 |     FROM ${_CROSS_OR_NATIVE}-builder AS builder
 202 | >>> RUN --mount=type=cache,target=${DOT_CACHE},sharing=locked,id=${SDK_NAME} \
 203 | >>>     --mount=type=cache,target=${BUILD_CACHE},sharing=locked,id=${SDK_NAME} \
 204 | >>>     install -v `swift build ${SWIFT_FLAGS} --show-bin-path`/poc-module-name-conflict /usr/bin
 205 |     
--------------------
ERROR: failed to solve: process "/bin/sh -c install -v `swift build ${SWIFT_FLAGS} --show-bin-path`/poc-module-name-conflict /usr/bin" did not complete successfully: exit code: 1
```

## How to enter docker building session:
Build with SLEEP=1
```bash
buildx build --load . --progress plain --build-arg SLEEP=1
```
Build session will hangs at sleep like following:
```log
#31 0.090 Enter the following command to enter build session:
#31 0.090 lima user:
#31 0.090     limactl shell docker bash -c 'sudo nsenter --all --target=$(lsns|awk "/^4026532477/{print \$4}") bash'
#31 0.090 
#31 0.090 Docker for Mac user:
#31 0.090     docker run -it --privileged --pid=host --rm ubuntu bash -c 'nsenter --all --target=$(lsns|awk "/^4026532477/{print \$4}") bash'
```
Copy and paste the command to enter build session.
```bash
$ docker run -it --privileged --pid=host --rm ubuntu bash -c 'nsenter --all --target=$(lsns|awk "/^4026532477/{print \$4}") bash'
################################################################
#                                                              #
# Swift Nightly Docker Image                                   #
# Tag: swift-DEVELOPMENT-SNAPSHOT-2024-06-13-a                 #
#                                                              #
################################################################
root@buildkitsandbox:/# 
```