# CLAUDE.md

## Project overview

This repository is a minimal Windows C++ console application for DLL-injection experiments. It combines a user-mode loader (`zhangbing_injector.cpp`) with an embedded signed kernel driver (`zhangbing_driver.h`) and several standalone driver artifacts.

This is security-sensitive research code. Treat every generated executable and every `.sys` file as a high-risk, untrusted binary.

## Safety and handling

- The generated injector and every `.sys` artifact are security-sensitive research binaries with a high-risk operational footprint.
- Routine work focuses on source review, static inspection, disassembly, signature/hash checks, and reproducible documentation.
- Dynamic research belongs in a disposable, isolated analysis environment with a documented containment and cleanup plan.
- Driver Signature Enforcement, test-signing settings, and Secure Boot remain at their normal secure values.
- A valid Authenticode signature establishes signing integrity; it does not establish that a driver or its use is safe.
- Uploading repository binaries to a scanning or sharing service requires explicit user permission.
- DLL injection and kernel-driver loading are reserved for explicitly authorized, controlled test environments.

## Repository layout

- `zhangbing_injector.cpp` — console entry point, driver service registration, process lookup, DLL loading, IOCTL request, and cleanup logic.
- `zhangbing_driver.h` — byte array containing the driver currently used by the injector.
- `Kernel.sys` — standalone copy of the driver embedded in `zhangbing_driver.h` at the current revision.
- `RainTrainerDriverV22_2.sys` — additional signed driver artifact; not referenced by the current project build.
- `RxDriver.sys` — additional signed driver artifact; not referenced by the current project build.
- `ZhangBing-Injector.sln` / `ZhangBing-Injector.vcxproj` — Visual Studio solution and project files.
- `README.md` — project description, translated into English.

## Important implementation behavior

The program:

- creates a random-looking driver filename under the user's temporary directory;
- enables debug and driver-loading privileges;
- creates and removes an `HKLM` service entry;
- loads a kernel driver through undocumented NT APIs;
- locates a process by executable name;
- reads a caller-provided DLL into memory;
- sends an injection request through `DeviceIoControl`;
- overwrites the dropped driver with random data before deleting it.

The embedded array currently has SHA-256:

`ad991cca59fd80192f445e0221a6e418137d9f0ddc48a12ef0335b0417329299`

This matches `Kernel.sys`. Keep the header array and standalone driver relationship explicit when either artifact changes.

## Build environment

- Visual Studio 2022 or Build Tools with the `v143` C++ toolset.
- Windows 10 SDK or a compatible installed Windows SDK.
- Configurations: `Debug|Win32`, `Release|Win32`, `Debug|x64`, and `Release|x64`.
- Unicode console application; no third-party package dependencies.

A build-only command is:

```powershell
msbuild "ZhangBing-Injector.sln" /t:Build /p:Configuration=Release /p:Platform=x64
```

The build output is an artifact for review; runtime execution is outside routine validation.

## Startup and use

The current implementation has **no safe default mode**. At a high level, the console asks for a DLL path and a target process name, drops and loads the embedded kernel driver, sends an injection request, and then attempts to unload and remove the driver. That description is for source review only; it is not a supported or safe runtime workflow.

For explicitly authorized dynamic research:

1. Use a disposable, isolated analysis system with no valuable data and no unrestricted network access.
2. Take a restorable snapshot before testing and document the containment and cleanup plan.
3. Use a benign test DLL and a disposable test process for analysis.
4. Capture the expected service, registry, driver-load, and `DeviceIoControl` behavior while keeping the payload and persistence surface limited to the documented test.
5. Revert the snapshot after testing and verify that no service or temporary driver remains.

A live command-line invocation is intentionally omitted because it would provide operational instructions for kernel-level arbitrary-code injection. A runnable teaching example can use clearly labeled mocks and an explicit dry-run mode in place of driver loading, unloading, and the IOCTL call.

## Working conventions

- Keep changes narrowly scoped; unrelated files remain unchanged.
- Preserve the public-domain/Unlicense status and existing attribution comments.
- Keep documentation in English unless the user requests another language.
- Treat the signed `.sys` artifacts as fixed binary data; regeneration, re-signing, replacement, and stripping are reserved for explicit requests.
- Treat the embedded driver bytes as binary data and preserve their byte-for-byte relationship with `Kernel.sys`.
- Prefer replacing risky runtime behavior with a clearly labeled no-op or simulation when creating tests or examples.
- The project scope excludes persistence, credential access, remote communication, and anti-security-product behavior.

## Validation

For documentation or source-only changes:

```powershell
git diff --check
git status --short
```

For an explicitly requested compile check, build the relevant configuration and report toolchain availability and warnings. The resulting binary remains outside routine validation.
