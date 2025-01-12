# Tutorial

## Installation

[**MeshIntegrals.jl**](https://github.com/JuliaGeometry/MeshIntegrals.jl) is
registered in the official Julia General registry, enabling it to be easily
installed using Julia on an internet-connected computer.
**MeshIntegrals.jl** should usually be installed in conjunction with
[**Meshes.jl**](https://github.com/JuliaGeometry/Meshes.jl) and
[**Unitful.jl**](https://github.com/PainterQubits/Unitful.jl) which provide
support for geometries and physical units, respectively.

**MeshIntegrals.jl** can be installed from the Julia REPL by entering `pkg` mode
with the `]` key and `add`ing it by name
```julia-repl
pkg> add MeshIntegrals
```

or by calling the built-in **Pkg.jl** library.
```julia
import Pkg
Pkg.add("MeshIntegrals")
```

## Basic Usage

Usage of **MeshIntegrals.jl** typically also involves using **Meshes.jl** and **Unitful.jl**,
so all three packages will be used in this example.

```@example tutorial
using Meshes
using MeshIntegrals
using Unitful
```

### Define a Geometry to Use as an Integration Domain

Here, representing the domain being integrated over, we will define a Bezier
curve whose path approximates a sine-wave on the xy-plane.

```@example tutorial
N = 361  # number of control points
xs = range(-π, π, length=N)  # x will be bounded in [-π, π] meters
curve = Meshes.BezierCurve([Point(x * u"m", sin(x) * u"m", 0.0u"m") for x in xs])
```

### Define an Integrand Function

Here, we will define an integrand function of the following form, where all
coordinates are defined in a physical space with meter units and the result is
a quantity in units of Ohms per meter.
```math
f(x, y, z) = \frac{1}{\sqrt{1 + \cos^2(x/\text{m})}} ~ \Omega/\text{m}
```

Integrand functions are expected to provide a method that takes a single
`Meshes.Point` argument, so this can be written in Julia as
```@example tutorial
function f(p::Meshes.Point)
    x, y, z = Meshes.to(p)
    return (1 / sqrt(1 + cos(x / u"m")^2)) * u"Ω/m"
end
```

Alternatively, this could be written in the user-friendly notation
```@example tutorial
f(x, y, z) = (1 / sqrt(1 + cos(x / u"m")^2)) * u"Ω/m"
f(p::Meshes.Point) = f(Meshes.to(p)...)
```
where the required `f(Meshes.Point)` method simply maps to the method `f(x, y, z)`.

### Integrating

This function can be integrated using recommended defaults simply by calling
```@example tutorial
integral(f, curve)  # -> Approximately 2π Ω
```

The alias function `lineintegral` works for this geometry since it has one
parametric dimension. However, the aliases `surfaceintegral` and `volumeintegral`
will throw an `ArgumentError` since the geometry is not a surface or volume.
```julia
lineintegral(f, curve)
# -> Approximately 2π Ω

surfaceintegral(f, curve)
# -> ERROR: ArgumentError: Performing a surface integral on a geometry with 1 parametric dimensions not supported.

volumeintegral(f, curve)
# -> ERROR: ArgumentError: Performing a volume integral on a geometry with 1 parametric dimensions not supported.
```

An `IntegrationRule` with settings can also be manually specified. The following
example uses the adaptive Gauss-Kronrod method with a loosened absolute tolerance
setting of $10^{-4}~\Omega$, which speeds up integration by sacrificing some
accuracy.
```@example tutorial
integral(f, curve, GaussKronrod(atol = 1e-4u"Ω")) # -> Approximately (2π ± 1e-4) Ω
```

The `integral` function and its aliases also support Julia's
[do-syntax](https://docs.julialang.org/en/v1/base/base/#do), which provides a
code block to define a single-use anonymous function and then injects it as a
first argument to the preceding call. This can be useful if the integrand
function will not be used outside this integration call.
```@example tutorial
integral(curve) do p
    x, y, z = Meshes.to(p)
    (1 / sqrt(1 + cos(x / u"m")^2)) * u"Ω/m"
end # -> Approximately 2π Ω
```
