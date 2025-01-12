# [Specializations](@id specializations)

There are several notable exceptions to how Meshes.jl defines [parametric functions](@ref how-parametric).
- `Meshes.ConeSurface` is essentially a composite type and has a parametric function that only maps the conical portion of the geometry, so the `Meshes.Disk` base element has to be integrated separately.
- `Meshes.CylinderSurface` is essentially a composite type and has a parametric function that only maps the cylindrical portion of the geometry, so the `Meshes.Disk` element has to be integrated separately.
- `Meshes.FrustumSurface` is essentially a composite type and has a parametric function that only maps the cylindrical portion of the geometry, so the top and bottom `Meshes.Disk` elements have to be integrated separately.
- `Meshes.Line` represents a line of infinite length that passes through two points, and it has a parametric function that is valid on the domain $(-\infty, \infty)$.
- `Meshes.Plane` represents a plane of infinite extent, and it has a parametric function that is valid on the domain $(-\infty, \infty)^2$.
- `Meshes.Ray` represents a line that begins at a point and extends in a particular direction with infinite length, and it has a parametric function that is valid on the domain $[0, \infty)$.
- `Meshes.Ring` is a composite type that lacks a parametric function, but can be decomposed into `Meshes.Segment`s and then integrated by adding together the individual integrals.
- `Meshes.Rope` is a composite type that lacks a parametric function, but can be decomposed into `Meshes.Segment`s and then integrated by adding together the individual integrals.
- `Meshes.Triangle` has a parametric function that takes coordinates on a 2D barycentric coordinate system. So, for `(::Meshes.Triangle)(t1, t2)`, the coordinates must obey: $t_1, t_2 \in [0,1]$ where $t_1 + t_2 \le 1$.
- `Meshes.Tetrahedron` has a parametric function that takes coordinates on a 3D barycentric coordinate system. So, for `(::Meshes.Tetrahedron)(t1, t2)`, the coordinates must obey: $t_1, t_2, t_3 \in [0,1]$ where $t_1 + t_2 + t_3 \le 1$.
