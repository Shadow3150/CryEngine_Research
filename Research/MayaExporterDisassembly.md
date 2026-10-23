# Maya exporter binary: code-level follow-up

Date: 2026-10-23
Target: MayaCryExport22017_64.mll (Windows x64 PE DLL)

## Static methods
Examined PE section layout, native imports, ASCII strings, and x86-64 disassembly with objdump. No Maya or RC execution was performed.

## Confirmed Maya skin API usage
The import table contains:
- `MFnSkinCluster::influenceObjects(MDagPathArray&, MStatus*)`
- `MFnSkinCluster::getWeights(MDagPath const&, MObject const&, MFloatArray&, unsigned int&)`
- `MFnBlendShapeDeformer::getBaseObjects` and `getTargets`
- `MFnAnimCurve` functions

This supports the conclusion that the native plugin reads Maya skinning and morph/animation information itself.

## Cross-reference investigation
PE `.text` VMA starts at `0x180001000` and file offset `0x400`; `.rdata` starts at `0x1800B9000` and file offset `0xB8400`.

The literal `EightWeightsPerVertex` is located at VA `0x1800BE300`, with code references at `0x18002920C` and `0x18002922F`. The surrounding instructions perform string handling, including copying the 21-character literal. **This does not yet establish whether the string is a setting, a COLLADA attribute, or how it controls the influence limit.**

`RC_Parameters` is located at VA `0x1800C5168`, referenced from `0x18006F194` and multiple other code sites (e.g. `0x180071EFB`). `rc.exe` is located at VA `0x1800C517C`, referenced at `0x18006F33A`. The surrounding code includes string assembly, but exact executable command line remains unverified.

## Important distinction
The diagnostics `too many skinning influences (maximum is %d)` show that a maximum is checked, but **the exact maximum, conditions, and behavior of EightWeightsPerVertex are not yet recovered**. Do not infer that eight weights are preserved end-to-end.

## MEL setup
`userSetup.mel` creates a CryMayaTools menu and sources `crytekSetup.mel`. `crytekSetup.mel` uses `CryToolsPath` / `CryLibsPath` Maya optionVars, adds module scripts/icons to search paths, and attempts to execute `CryMayaTools/CryCore/startup.py`. Default internal paths reference `\\fs1\TECHART\Tools\`, so the MEL scripts alone are not a self-contained exporter.

## Next experiments
1. Follow the call chain around `EightWeightsPerVertex` to determine the value's source and consumer.
2. Trace RC executable and argument construction, including user-supplied `RC_Parameters`.
3. Acquire a known-good Maya-exported DAE (or produce a minimal independent test DAE) to compare skin controller layout and Crytek-specific extras.
4. Validate compilation with the user's actual RC version and inspect compiled skinning data before making any changes to Blender skin weights.
