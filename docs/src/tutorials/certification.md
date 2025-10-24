# Using SymbolicAnalysis.jl for convexity certificates

In this tutorial, we will show how to use automatic convexity certification of the optimization problem using [SymbolicAnalysis.jl](https://github.com/Vaibhavdixit02/SymbolicAnalysis.jl).

This works with the `structural_analysis` keyword argument to `OptimizationProblem`. This tells the package to try to trace through the objective and constraints with symbolic variables (for more details on this look at the [Symbolics documentation](https://symbolics.juliasymbolics.org/stable/manual/functions/#function_registration)). This relies on the Disciplined Programming approach hence neccessitates the use of "atoms" from the SymbolicAnalysis.jl package.

We'll use a simple example to illustrate the convexity structure certification process.

```@example symanalysis
using SymbolicAnalysis, Zygote, LinearAlgebra, Optimization, OptimizationLBFGS

function f(x, p = nothing)
    return exp(x[1]) + x[1]^2
end

optf = OptimizationFunction(f, Optimization.AutoForwardDiff())
prob = OptimizationProblem(optf, [0.4], structural_analysis = true)

sol = solve(prob, LBFGS(), maxiters = 1000)
```

The result can be accessed as the `analysis_results` field of the solution.

```@example symanalysis
sol.cache.analysis_results.objective
```

Relatedly you can enable structural analysis in Riemannian optimization problems (supported only on the SPD manifold).

We'll look at the Riemannian center of mass of SPD matrices which is known to be a Geodesically Convex problem on the SPD manifold.

```@example symanalysis
using Optimization, OptimizationManopt, Symbolics, Manifolds, Random, LinearAlgebra,
      SymbolicAnalysis

M = SymmetricPositiveDefinite(5)
m = 100
σ = 0.005
q = Matrix{Float64}(LinearAlgebra.I(5)) .+ 2.0

data2 = [exp(M, q, σ * rand(M; vector_at = q)) for i in 1:m];

f(x, p = nothing) = sum(SymbolicAnalysis.distance(M, data2[i], x)^2 for i in 1:5)
optf = OptimizationFunction(f, Optimization.AutoZygote())
prob = OptimizationProblem(optf, data2[1]; manifold = M, structural_analysis = true)

opt = OptimizationManopt.GradientDescentOptimizer()
sol = solve(prob, opt, maxiters = 100)
```
