# Archive method and verification

[README](../README.md) · [Installation guide](install.md)

This archive contains application packages and assets extracted from BUG Clicker's Windows, Linux, and macOS distributions. It does not contain the complete Electron runtime. Some distributed JavaScript is minified; the archive is not presented as the developer's original editable project or as a complete set of source maps.

## Extraction and REA analysis

For Windows, the installer was opened with `tar` to obtain `resources/app.asar`, and the ASAR header, file offsets, and sizes were used to unpack its application files. The Linux AppImage and macOS disk image likewise supplied their respective original ASAR packages and support files. No changes were made to the extracted application files.

[REA: Reverse Engineer Anything](https://github.com/morluto/rea) **6.8.0** was used for a subsequent static analysis of the Windows JavaScript application via `analyze-javascript-application`. REA was not the tool that extracted the Windows ASAR. Its semantic graph reached configured node limits, so its coverage was partial; the analysis does not prove that unobserved behavior is absent. REA was not used here to claim equivalent analysis of the Linux or macOS builds.

## Package inventory

The extracted file counts were compared with the corresponding ASAR entries by size and SHA-256. All three original installers identified Electron **44.2.0**; it must be installed separately to run the extracted app.

| Platform | Original app | Extracted files | Original ASAR SHA-256 |
| --- | --- | ---: | --- |
| Windows x64 | 1.19.2 | 129 | `33a3e46ea69e250756887ef1c7ebb82c126e148c178723a751599deba56ae7a6` |
| Linux x64 | 1.0.0 | 129 | `8db25fab38366f5295410c89194325bb7fe66fba110f5a46c863e58f870a8d09` |
| macOS x64 / arm64 | 1.4.2 | 128 | `e03e7facc6bb9b2255b0686522e0c1cd66e504789a99c2c4b91123edf89c5a45` |

The platform builds differ, so use each operating system's own files. Their original update configuration, notice handling, and links to the creator's services remain intact. The original application's author is identified as BUG; the Windows and Linux updates refer to [BUG-INVADE/Bug-Clicker-download](https://github.com/BUG-INVADE/Bug-Clicker-download), and the Mac updates refer to [BUG-INVADE/Bug-Clicker-forMAC-download](https://github.com/BUG-INVADE/Bug-Clicker-forMAC-download).

## Execution limits

The original Windows app was smoke-tested with Electron 44.2.0 in an isolated profile; Linux x64 was also smoke-tested using WSL2. These checks do not establish identical behavior for every feature or host. Actual macOS execution, signing, Gatekeeper, and Input Monitoring were not tested. Running an extracted app in development Electron can differ from the installed original in packaging, paths, signing, updates, and OS integration. See the [installation guide](install.md) for platform-specific dependencies and limits.
