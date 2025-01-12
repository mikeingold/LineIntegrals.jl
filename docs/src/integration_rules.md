# Integration Rules

An **integration rule** is a method or algorithm used to numerically calculate the value of an integral. When an integral is calculated using the two-argument form `integral(f, geometry)`, **MeshIntegrals.jl** will automatically select an integration rule. Default rules are generally well-behaved, but may not be optimal for every problem.

**MeshIntegrals.jl** defines an abstract type `IntegrationRule` with sub-types representing the various integration rules supported by this package. These rules can be specified when calculating an integral using the three-argument form `integral(f, geometry, rule)`. Currently, the following rule types are implemented:
- [`GaussKronrod`](@ref gausskronrod) for adaptive Gauss-Kronrod quadrature rules
- [`GaussLegendre`](@ref gausslegendre) for Gauss-Legendre quadrature rules
- [`HAdaptiveCubature`](@ref hadaptivecubature) for h-adaptive cubature rules

## [Gauss-Kronrod](@id gausskronrod)

The `GaussKronrod` type is used to specify an adaptive Gauss-Kronrod quadrature rule, as implemented by [QuadGK.jl](https://github.com/JuliaMath/QuadGK.jl).

```julia
rule = GaussKronrod()
```

All standard `QuadGK.quadgk` keyword-argument options are supported. These can be specified when constructing the rule, where the `kwargs` in `GaussKronrod(kwargs...)` is equivalent to `quadgk(f, a, b; kwargs...)`, e.g.:
```julia
rule = GaussKronrod(order = 5, rtol = 1e-4)
```

## [Gauss-Legendre](@id gausslegendre)

The `GaussLegendre` type is used to specify a [Gauss-Legendre quadrature](https://en.wikipedia.org/wiki/Gauss%E2%80%93Legendre_quadrature) rule. Gauss-Legendre quadrature rules of order $N$ are used to approximate definite integrals by sampling the integrand on a fixed grid with corresponding nodes $x_i$ and weights $w_i$.
```math
\int_{-1}^1 f(x) ~\text{d}x \approx \sum_{i=1}^N w_i \, f(x_i)
```

These nodes and weights are purely a function of $N$ as they are derived from the roots of $N$-th order Legendre polynomials. This has the effect of providing exact solutions for integrand functions $f$ that can be represented as degree $2N-1$ polynomials.

This integration process can also be extended into multiple dimensions, for example:
```math
\int_{-1}^1 \int_{-1}^1 \int_{-1}^1 f(x, y, z) ~\text{d}x ~\text{d}y ~\text{d}z \approx \sum_{i=1}^N \sum_{j=1}^N \sum_{k=1}^N w_i\,w_j\,w_k \, f(x_i, y_i, z_i)
```

MeshIntegrals.jl uses [FastGaussQuadrature.jl](https://github.com/JuliaApproximation/FastGaussQuadrature.jl) to efficiently compute Gauss-Legendre quadrature nodes and weights at the time a `GaussLegendre` rule is constructed. Once constructed, these rules can be re-used to calculate multiple integrals to improve performance. Additionally, MeshIntegrals.jl uses an allocation-free summation routine that further improves performance by avoiding storing intermediate results.
```julia
rule = GaussLegendre(N)
```

By contrast to adaptive integration rules where compute times can sometimes vary significantly, this technique has the advantage that compute times are much more predictable. For example, calculating a one-dimensional integral whose integrand $f$ can be computed in 10 microseconds using a `GaussLegendre(100)` can be expected to take approximately 1 millisecond. However, this lacks many of the guard-rails present in adaptive routines: results are not automatically checked to ensure convergence, so care must be taken to ensure that an appropriate rule and order are chosen.

## [H-Adaptive Cubature](@id hadaptivecubature)

The `HAdaptiveCubature` type is used to specify an h-adaptive cubature rule, as implemented by [HCubature.jl](https://github.com/JuliaMath/HCubature.jl).

```julia
rule = HAdaptiveCubature()
```

All standard `HCubature.hcubature` keyword-argument options are supported. These can be specified when constructing the rule, where the `kwargs` in `HAdaptiveCubature(kwargs...)` is equivalent to `hcubature(f, a, b; kwargs...)`, e.g.:
```julia
rule = HAdaptiveCubature(order = 5, rtol = 1e-4)
```
