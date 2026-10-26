# SpatialGrid

Reusable grid coordinates, chunk indexing, and neighborhood queries.

> Planning repository — documentation and Git initialization only. No application, package, solution, database, tests, CI workflow, or runnable demo has been implemented yet. Everything below describes the proposed implementation, unless explicitly marked as verified.

## Goals

Provide predictable integer spatial primitives for simulations and voxel worlds, with explicit boundaries and tested behavior around negative coordinates and chunk seams.

## Target users

Developers building tile simulations, cellular automata, agent worlds, and chunked voxel sandboxes.

## MVP scope

- Immutable Int2/Int3 coordinates, bounds, and checked coordinate conversions.
- Dense bounded 2D grid and sparse chunked 3D cell storage with documented memory behavior.
- Von Neumann/Moore neighborhood enumeration in 2D and face/box neighborhoods in 3D.
- Explicit reject, clamp, and wrap boundary policies.
- World-to-chunk/local mapping with floor division and deterministic iteration.

## Stretch goals

- Spatial hash for continuous-position proximity queries as a separate API.
- Occupancy masks, pooling, and cache-aware traversal based on benchmarks.
- Optional pathfinding utilities only after storage/query contracts stabilize.

## Proposed tech stack

C# / .NET library with a compatibility target selected jointly with ForgeMatter's Godot version; net10.0 is the default portfolio target, but this library may multi-target an older compatible TFM if required. xUnit and BenchmarkDotNet; no engine types or rendering dependency.

Select compatible stable dependency versions during milestone 1, record them in project files, and pin the SDK/toolchain after the compatibility spike. No packages or development workloads are installed by this documentation task.

## Architecture

Coordinates and bounds are immutable value types. Storage implementations expose common read/write contracts while retaining separate dense/sparse cost models. Neighborhood functions operate on coordinates and explicit boundary policies. Chunk codecs are independent from persistence file formats. Enumeration must not mutate collections or expose backing arrays accidentally.

## Proposed folder structure

Only README.md and .gitignore exist initially, alongside Git metadata. The following tree is a plan; create directories when their implementation begins.

```text
SpatialGrid/
  README.md
  .gitignore
  SpatialGrid.slnx
  src/SpatialGrid/
  tests/SpatialGrid.UnitTests/
  tests/SpatialGrid.PropertyTests/
  benchmarks/SpatialGrid.Benchmarks/
  samples/SpatialGrid.ConsoleDemo/
  docs/coordinates/
```

## Data models and invariants

Int2(X,Y); Int3(X,Y,Z); Bounds2(MinInclusive, MaxExclusive); Bounds3; ChunkKey(Int3); ChunkLayout(Size); BoundaryMode; DenseGrid2<T>; ChunkGrid3<T>. Missing sparse cells return an explicit configured default; distinguish absent chunks from allocated chunks filled with default values when reporting memory.

## Proposed API and contracts

Proposed API: grid.TryGet(position, out value); grid.Set(position, value); Neighborhood.Moore2(position, radius, bounds, boundaryMode); layout.Split(worldPosition) -> (chunk, local); layout.Combine(chunk, local) -> world. For size 16, world X=-1 maps to chunk X=-1 and local X=15. Specify duplicate suppression when wrapping tiny dimensions.

Version persisted/exported formats from their first implementation. Validate at boundaries and return structured errors rather than leaking internal exceptions. The contracts above remain proposals until backed by code and contract tests.

## Implementation milestones

1. Write coordinate/boundary specification and negative-coordinate examples.
2. Implement dense 2D storage and neighborhood queries for EcoGenesis.
3. Implement chunked 3D storage and seam queries for ForgeMatter.
4. Benchmark occupancy patterns, validate engine compatibility, and produce package smoke tests.

Each milestone should end with a small demonstrable result, a written acceptance check, and tests for its important failure modes. Work through a vertical slice before expanding the stretch scope.

## Testing and CI plan

Property-test Split/Combine round trips including negative coordinates, boundary wrapping, neighborhood symmetry/counts, tiny wrapped dimensions, integer overflow, and deterministic iteration. Compare dense/sparse results on the same finite world. CI builds/tests all declared TFMs on Windows/Linux, packs without publishing, and validates sample consumers.

CI is planned, not configured. Once a solution exists, add .github/workflows/ci.yml with read-only default permissions, explicit toolchain setup, dependency restore, build, and the checks above. Keep ordinary CI offline from real services and hardware. Store test reports as artifacts; do not publish packages, create releases, or deploy from the first workflow. Record hardware/manual results separately from automated passes.

## Setup and development

### Current checkout

Open this README in a Markdown viewer. Git is initialized locally; no build or run step exists yet. Check the working tree with `git status --short --branch`. Documentation files start untracked; no initial commit or remote is created.

### Prerequisites and first implementation

After scaffolding, choose target frameworks using a Godot compatibility spike rather than assuming .NET 10 works in every engine release. Build the coordinate primitives first, then dense storage, then sparse chunks. Use the console demo to visualize small coordinate maps before integrating either application.

Install only the dependencies needed for the selected milestone. Add a checked-in sample configuration with harmless defaults; keep local databases, recordings, generated datasets, credentials, and device identifiers outside version control. Pick a license before public distribution; no license is implied by this plan.

### Future commands

These commands are examples for after the referenced projects/files have been created; they will not build this documentation-only repository.

```powershell
dotnet restore SpatialGrid.slnx
dotnet test SpatialGrid.slnx
dotnet pack src/SpatialGrid -c Release -o artifacts/packages
```

## Supporting libraries and boundaries

Used by EcoGenesis for 2D neighborhoods and ForgeMatter for chunked 3D indexing. Keep both engine and application dependencies out of this library.

Consume stable library versions through local package artifacts initially. Avoid absolute ProjectReference paths into sibling repositories and avoid Git submodules for the MVP. Keep a consumer's first vertical slice runnable using a small local interface or fixture while a supporting package is under development.

## Realistic limitations

No automatic thread safety, GPU storage, octree, geographic projection, collision engine, or general physics in the MVP. Sparse storage still becomes expensive at high occupancy; benchmark before choosing it. Coordinate overflow and pathological allocation requests must fail predictably.

Scope and performance targets are hypotheses until measured. Document reproducible dataset sizes, hardware, dependency versions, and failure cases alongside any benchmark claim. Do not present planned capabilities as shipped features.

## Documentation provenance

Prepared 2026-10-23. The previous chat supplied the project names and the AutoDiag platform change, but its detailed assistant responses were unavailable content references. The user confirmed the seven proposed application themes in this session; individual designs here are newly proposed plans, not a recovered transcript.

