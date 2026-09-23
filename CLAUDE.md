# CLAUDE.md

## Project overview

This repository is a minimal Windows C++ console application for DLL-injection experiments. It combines a user-mode loader (`zhangbing_injector.cpp`) with an embedded signed kernel driver (`zhangbing_driver.h`) and several standalone driver artifacts.

This is security-sensitive research code. Treat every generated executable and every `.sys` file as a high-risk, untrusted binary.

## Safety rules

- Do not execute the generated injector on a normal or production Windows host.
- Do not load, install, map, debug, or manually map any repository driver.
- Do not inject a DLL into another process as part of testing or automation.
- Do not weaken Driver Signature Enforcement, enable test-signing mode, or change Secure Boot policy.
- A valid Authenticode signature proves signing integrity, not that a driver or its use is safe.
- Do not upload repository binaries to a scanning or sharing service without explicit user permission.
- Prefer static inspection, disassembly, signature/hash checks, and source review.
- If dynamic analysis is explicitly requested, require a disposable, isolated analysis environment and document the containment plan first.

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

Building is not the same as authorizing execution. Never run the output as part of routine validation.

## Startup and use

The current implementation has **no safe default mode**. At a high level, the console asks for a DLL path and a target process name, drops and loads the embedded kernel driver, sends an injection request, and then attempts to unload and remove the driver. That description is for source review only; it is not a supported or safe runtime workflow.

For any explicitly authorized dynamic research:

1. Use a disposable, isolated analysis system with no valuable data and no unrestricted network access.
2. Take a restorable snapshot before testing and document the containment and cleanup plan.
3. Use only a benign test DLL and a disposable test process, never a production or third-party process.
4. Capture the expected service, registry, driver-load, and `DeviceIoControl` behavior for analysis; do not broaden the payload or add persistence.
5. Revert the snapshot after testing and verify that no service or temporary driver remains.

Do not publish or follow a live command-line invocation for the real injector in this file. It would provide operational instructions for kernel-level arbitrary-code injection. For a runnable teaching example, first replace driver loading, unloading, and the IOCTL call with clearly labeled mocks and add an explicit dry-run mode.

## Working conventions

- Keep changes narrowly scoped and do not modify unrelated files.
- Preserve the public-domain/Unlicense status and existing attribution comments.
- Keep documentation in English unless the user requests another language.
- Do not regenerate, resign, replace, or strip the signed `.sys` artifacts casually.
- Treat the embedded driver bytes as binary data; do not reformat or partially edit the byte array.
- Prefer replacing risky runtime behavior with a clearly labeled no-op or simulation when creating tests or examples.
- Never add persistence, credential access, remote communication, or anti-security-product behavior.

## Validation

For documentation or source-only changes:

```powershell
git diff --check
git status --short
```

For an explicitly requested compile check, build the relevant configuration but do not execute the resulting binary. Report toolchain availability and any warnings accurately.
