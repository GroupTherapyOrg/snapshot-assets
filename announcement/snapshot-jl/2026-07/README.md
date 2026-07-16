# [ANN] Snapshot.jl — interactive Pluto exports without a Julia server

Hi!

I have been working on **[Snapshot.jl](https://github.com/GroupTherapyOrg/Snapshot.jl)**, a package that exports a Pluto notebook as a static page while keeping supported `@bind` interactions live.

The interactive cells are compiled to WebAssembly with [WasmTarget.jl](https://github.com/GroupTherapyOrg/WasmTarget.jl). When somebody moves a slider, the affected Julia code runs in their browser. The exported page does not need a Julia server, a kernel, or precomputed slider responses.

[**Open the live notebook gallery →**](https://grouptherapyorg.github.io/Snapshot.jl/notebooks/)

![The Snapshot.jl documentation homepage and quickstart](https://raw.githubusercontent.com/GroupTherapyOrg/snapshot-assets/main/announcement/snapshot-jl/2026-07/01-gallery-highlighted.png)

## A small example

Install Snapshot.jl from the General registry:

```julia
import Pkg
Pkg.add("Snapshot")

using Snapshot

export_notebook("notebook.jl"; therapy=true)
```

This runs the notebook and writes a lean, self-contained Therapy component: server-rendered cell output plus a small WebAssembly island for each reactive group that compiled and passed verification.

The classic Pluto-style HTML export is also available:

```julia
export_notebook("notebook.jl")
```

Both outputs are static files and can be served by an ordinary static host.

## What happens during export?

Snapshot does four things:

1. Runs the notebook once in Pluto.
2. Uses Pluto's dependency information to find cells affected by each `@bind` value.
3. Lifts those reactive groups into Julia functions and compiles their typed IR to WasmGC through WasmTarget.jl.
4. Checks the browser result against the real notebook before including the island.

![The Snapshot.jl extraction, compilation, verification, and rendering pipeline](https://raw.githubusercontent.com/GroupTherapyOrg/snapshot-assets/main/announcement/snapshot-jl/2026-07/02-how-it-works.png)

If a group does not compile or does not reproduce the notebook result, it does not quietly ship a different answer. The original output stays on the page and the export explains that the interaction is static. A notebook can therefore be partly interactive without being treated as a failed export.

## Some notebooks

The documentation currently contains 16 lightly adapted Pluto featured notebooks. It reports coverage per notebook and links back to the `.jl` source.

![The Snapshot.jl featured-notebook gallery with per-notebook interactive-cell counts](https://raw.githubusercontent.com/GroupTherapyOrg/snapshot-assets/main/announcement/snapshot-jl/2026-07/03-notebooks.png)

Here is a chemistry notebook. Its three reactive cells compile, so the titration curve is recalculated and plotted in the browser as the inputs change.

[**Open Simulating titrations →**](https://grouptherapyorg.github.io/Snapshot.jl/notebooks/Titration/)

![A chemistry notebook exported by Snapshot.jl](https://raw.githubusercontent.com/GroupTherapyOrg/snapshot-assets/main/announcement/snapshot-jl/2026-07/04-titration.png)

The output is still a notebook: Markdown, equations, Julia source, controls and figures remain in document order. The lean export removes the Pluto editor and session state rather than redesigning the notebook as an application.

![A live image-filtering notebook with a PlutoUI slider and WasmMakie output](https://raw.githubusercontent.com/GroupTherapyOrg/snapshot-assets/main/announcement/snapshot-jl/2026-07/05-convolution.png)

The examples include larger reactive graphs too. This fractals notebook currently has 15 of 15 interactive cells compiled.

[**Open Fractals and Fractal Art →**](https://grouptherapyorg.github.io/Snapshot.jl/notebooks/fractals/)

![The Fractals and Fractal Art notebook exported as a lean page](https://raw.githubusercontent.com/GroupTherapyOrg/snapshot-assets/main/announcement/snapshot-jl/2026-07/06-fractals.png)

Canvas output uses [WasmMakie.jl](https://github.com/GroupTherapyOrg/WasmMakie.jl). The visible canvas remains mounted while a complete new frame is drawn and presented, so rapid slider changes do not replace the figure with an intermediate blank canvas.

![A turtle-art notebook drawing to a browser canvas through WasmMakie](https://raw.githubusercontent.com/GroupTherapyOrg/snapshot-assets/main/announcement/snapshot-jl/2026-07/07-turtles.png)

## The current boundary

Snapshot.jl is an exporter, not a Julia interpreter in the browser. Its interactive coverage is the subset that WasmTarget.jl can compile faithfully from type-inferred Julia IR.

That subset already includes ordinary control flow, closures, arrays, strings, dictionaries, exceptions, a growing part of Base and several numerical packages. It is not all of Julia. Dynamic dispatch, runtime facilities, `ccall`-dependent paths and unsupported library internals can still stop an island from compiling.

This boundary is visible on purpose. The gallery reports how many cells are interactive, and failed groups keep their build-time output. As WasmTarget.jl matures, Snapshot.jl can compile more notebooks without changing the notebook format or the hosting model.

## Status and trust boundary

- Snapshot.jl is pre-1.0 and available from the General registry with `Pkg.add("Snapshot")`.
- It currently targets Julia 1.12 because it works directly with Julia compiler IR.
- The package is MIT licensed. WasmTarget.jl is Apache-2.0 licensed.
- Exporting **executes the notebook** with the permissions of the Julia process. It is not a sandbox; only export notebooks and environments you trust.
- The generated page contains static HTML, JavaScript glue and WebAssembly. It does not contain an always-running Julia process.

The package source, tests and current limitations are in the **[Snapshot.jl repository](https://github.com/GroupTherapyOrg/Snapshot.jl)**. I would be especially interested in notebooks that export correctly but leave an interaction static, because those tend to be useful, concrete WasmTarget test cases.

## One small preview

I am also testing **[snapshot.show](https://snapshot.show)**, a separate hosted workflow that builds Snapshot.jl exports from GitHub repositories. I will write about that service separately after the package has had some time on its own.
