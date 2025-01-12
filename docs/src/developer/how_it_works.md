# How it Works (By Example)

## Example Problem

Let $f$ be a function of position $\bar{r}$ in some space.
```julia
function f(r̄::Meshes.Point)
    x, y, z = to(r̄)
    ...
end
```

Let the integration domain be the space (a ball) enclosed by a sphere centered on the origin with a radius of 5 meters.
```julia
center = Meshes.Point(0u"m", 0u"m", 0u"m")
radius = 5.0u"m"
ball = Meshes.Ball(center, radius)
```

This integral is often expressed abstractly as simply the following, where the triple integral signs and $\text{d}V$ indicate that the integration domain is some three-dimensional volume.
```math
\iiint f(\bar{r}) ~ \text{d}V
```

Integrals like this are often solved manually by selecting an appropriate coordinate system and limits that neatly represent the integration domain, e.g.
```math
\int_0^{\pi} \int_0^{2\pi} \int_0^{5} f(\bar{r}) ~ \text{d}\rho~\text{d}\theta~\text{d}\phi
```

This works great for simple geometries, but requires integration code that is geometry-specific. This package leverages parametric functions defined in Meshes.jl and differential forms to define integral methods that are general solutions for all geometries.

## [Parametric Functions](@id how-parametric)

Every supported `Meshes.Geometry` type is defined as having a parametric function that maps from a local parametric coordinate system to every point on the geometry. Curve-like geometries will have a single parametric dimension, surfaces will have two dimensions, and volumes will have three dimensions; this can be checked for a particular geometry via `Meshes.paramdim(geometry)`.

For consistency across geometry types, with [some notable exceptions](@ref specializations), these parametric functions are defined to take coordinates inside a normalized range $[0,1]$. In the example case of `ball`, Meshes.jl defines a parametric function mapped in normalized spherical coordinates $(t_\rho, ~t_\theta, ~t_\phi)$. We find, then:
```julia
Meshes.paramdim(ball) == 3    # a volume

ball(tρ, tθ, tφ)    # for args in range [0, 1], maps to a corresponding Meshes.Point

ball(0, tθ, tφ) == center
```

In effect, we can now use the geometry itself as a function that maps from three normalized ($0 \le t \le 1$) arguments to every point on the geometry. For the sake of generalization, let this parametric function be called $g$.
```math
\text{g}: (t_1,~t_2,~t_3) ~\mapsto~ \text{Point}\big[ x, ~y, ~z \big]  
```

## Differential Forms

Using differential forms, the general solution for integrating a geometry with three parametric dimensions ($t_1$, $t_2$, and $t_3$) is
```math
\iiint f(r̄) ~ \text{d}V = \iiint f(\bar{r}) ~ \bar{\text{d}t_1} \wedge \bar{\text{d}t_2} \wedge \bar{\text{d}t_3}
```

This resultant differential (volume) element is formed at each point in the integration domain by taking [the Jacobian](https://en.wikipedia.org/wiki/Jacobian_matrix_and_determinant) of the parametric function.
```math
\mathbf{J}_f = \begin{bmatrix} \bar{\text{d}t_1} & \bar{\text{d}t_2} & \bar{\text{d}t_3} \end{bmatrix}
```
where
```math
\bar{\text{d}t_n} = \frac{\partial}{\partial t_n} ~ \text{g}(t_1,~t_2,~t_3)
```

Each of these partial derivatives is a vector representing the direction that changing each parametric function argument will move the resultant point. The differential element ($E$) size is then calculated using geometric algebra as the magnitude of the exterior product ($\wedge$) of these three vectors.
```math
E(t_1,~t_2,~t_3) = \left\| \bar{\text{d}t_1} \wedge \bar{\text{d}t_2} \wedge \bar{\text{d}t_3} \right\|
```

Finally, we use the parametric function itself, $g$, as a map to all points $\bar{r}$ in the integration domain. Since `Meshes.Geometry` parametric functions all operate on normalized domains, we can now solve any volume integral as simply
```math
\int_0^1 \int_0^1 \int_0^1 f\Big(\text{g}\big(t_1,~t_2,~t_3\big)\Big) ~ E(t_1,~t_2,~t_3) ~ \text{d}t_1 ~ \text{d}t_2 ~ \text{d}t_3
```

This form of integral can be trivially generalized to support $n$-dimensional geometries in a form that enables the use of a wide range of numerical integration libraries.
