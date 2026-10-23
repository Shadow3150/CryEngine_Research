# MayaCryExport22017_64.mll — static analysis (2026-10-23)

## Evidence and scope
Analyzed the user-provided binary `MayaCryExport22017_64.mll` without running Maya. Windows x86-64 PE DLL, 1,111,552 bytes. The binary contains the string `Exporter build created on May 18 2022` (self-reported build date).

## Confirmed embedded strings
- `EightWeightsPerVertex`, `UseF32VertexFormat`, `UseCustomNormals`, `DoNotMerge`, `VClothPreProcess`
- `ExportCompressedCOLLADA`, `.dae.zip`, `.dae`, `Generating intermediate file (*.dae).`, `Compiling intermediate file to engine format.`
- `RC_Parameters`, `rc.exe`, ` /userdialog=0 `, ` /userdialog=1 `, `ResourceCompiler executable was not found.`
- `Vertex weights don't add up to 1. (%f)`
- `%d vertices have too many skinning influences (maximum is %d).`
- `Mesh `%s` has %I64u vertices with too many skinning influences (maximum is %d).`
- `meshChrNotSkinned`, `meshSkinnedNotInChr`, `skinBoundToNonJoint`
- `cryExportNode`, `cryExportType`, `skeletonRootFromExportNode`

## Binary offsets (file offsets, hexadecimal)
- `EightWeightsPerVertex`: `0xBD700`
- `UseF32VertexFormat`: `0xBD730`
- `ExportCompressedCOLLADA`: `0xBBC60`
- `RC_Parameters`: `0xC4568`
- `rc.exe`: `0xC457C`

## Dependency clues
PE import table includes `OpenMaya.dll`, `OpenMayaAnim.dll`, `OpenMayaUI.dll`, and `Foundation.dll`, consistent with a native Maya plugin.

## Hypotheses (not yet verified)
- `EightWeightsPerVertex` may toggle the skinning influence limit between four and eight; a string alone does not prove behavior.
- The plugin appears to produce COLLADA as an intermediate and invoke RC; exact command construction and argument order still require disassembly or runtime traces.
- Whether all eight weights survive the intermediate file and RC compilation is unknown.

## Next investigations
1. Locate cross-references to `EightWeightsPerVertex` and identify the validation branch and maximum-influence value.
2. Recover RC command construction around `RC_Parameters`, `rc.exe`, and `/userdialog`.
3. Compare a minimal COLLADA with 4 and 8 influences per vertex, ideally against an actual Maya-generated sample.
4. Preserve the original Blender character; do not reduce weights without evidence.
