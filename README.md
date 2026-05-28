# Lagrangian Dynamics Script

This script will model any classically dynamic system using the calculus of variations. It calculates the kinetic and potential energy in an arbitrary coordinate system and prepares the constrained | unconstrained Euler Lagrange equations with a formatted return string.

---

## DERIVATIVES

The computation assumes that all total derivatives are total *time* derivatives. We assume that the coordinates which describe the system are independent of each other in order to construct the Euler Lagrange equations anyhow, so this assumption isn't bad. As such, Dt[anything] shows up in dot notation.

---

## CORE FUNCTIONS

There is a closed cell grouped under the `ClearAll` block. You can expand it by clicking on it's tiny bookend, dropping down the cell menu from the top, hovering cell properties, and checking open. This cell group has the functions for the Euler Lagrange equations and chain rule integration pattern matching. Either way, just make sure that if you clear Global, that you click that tiny bookend and run the closed (or open if you leave it open) section with these crucial functions so they don't go away with everything else.

---

## PURE FUNCTIONS

The functions at the very beginning are QOL. Any function with `#`'s is a pure function, where `#` is a placeholder. These use `&`, a variadic function, to defer binding input to `#`. Exploring these definitions, print the leftmost block before the first `&`, then read them from right to left to consider the layers of the transformation.

For the simplest function `foo:=#&`, calling `foo[x]` trivially returns `x`. Calling `foo[{x,y}]` returns `{x,y}`. `#` is a very interesting semantic. It can be used in typical expressions, like `Integrate[tough_integrand,{arbitrary_bounds}]` rewritten into `Integrate[#,{arbitrary_bounds}]&[tough_integrand]`. This rewriting may ergonomically position commonly editted expressions. In most cases, a rightmost positioning is conveniently accessible.

In the case of `foo[{x,y}]`, the `#` has an index at `[[1]]` and `[[2]]` such that:

```mathematica
foo := #[[2]] &
foo[{x, y}]
```
returns `y`.

Writing,
```mathematica
foo := # &
foo[x, y]    (* not a vector input *)
```
returns `x`. To return `y`, define `foo` as `#2&` (not `#[[2]]`). It seems that `#`'s binds eagerly on the first argument before returning.

Use `##` for nonscalar returns | multi-assign: `##&` returns arguments without structuring them. `{##}&` returns them as a vector.

---

### EXAMPLES

*Note (naming convention): Querying functions are named with a closing Q.*

```mathematica
Mag := If[VectorQ[#], Sqrt[# . #], #] &;
```

Calling `Mag[x]` transforms an arbitrary `x` via `If[VectorQ[x], Sqrt[x . x], x]`, useful in debugging inputs for plotting, for instance.

These compose.

```mathematica
QA := Table[#1, {#2, #3}] &[#1, If[AnyMatch[{1, Null}, #1], i, Vars[#1][[1]]], #2] &;
```

Calling `QA[c,3]` returns the vector `{1,2,3}`, just as `Table[c,{c,3}]` would. `QA[c^2,3]` returns `{1,4,9}`, `QA[D[Cos[t], {t, n}], 3]={-Sin[t], -Cos[t], Sin[t]}` (iteration depends on alphabetic ordering)

Since `Vars` returns a vector, the first index (`[[1]]`) is selected in `Vars[#1][[1]]` for trivial polymorphism. It's not clean, but it's easier to iterate with.

`QA` is convenient for experimentation.

```mathematica
QA2 := {##} &;
QA2[1, 3]

QA2 := {##} &[#1, If[AnyMatch[{1, Null}, #1], i, Vars[#1][[1]]], #2] &;
QA2[1, 3]

QA2 := Table[#1, {#2, #3}] &[#1, If[AnyMatch[{1, Null}, #1], i, Vars[#1][[1]]], #2] &;
QA2[1, 3]
```

All in one evaluation block. This trivializes observability over input arguments evolution through partially applied blocks | transformation layers.

`@@` folds. `Times@@{##}/.#1->1` substitutes the first index to `1`, then applies a multiplicative fold across the input vector.

`S` is a quick `simplify`:

```mathematica
S := If[#2 === 3, #1 /. g21[], #1] &[If[AnyMatch[{2, 3}, #2], FactorTerms[#1, CsQ[#1]], #1], #2] &[If[AnyMatch[{0, Null}, #2], #1,Refine[FullSimplify[#1], Variables[#1] \[Element] Reals] /. Abs[q_] :> q], #2] &[#1, (Times @@ ({##} /. #1 -> 1))] &;
```

`S` takes an optional parameter such that the degree of simplification is ergonomically configurable. Leaving out the optional parameter simply sets it to a default, `#1 -> 1`. Passing `0`, or even empty (`S[x,]`), shorts the function such that the expression is not simplified. The flow is in writing `S[x]` and flickering `S[x,]` to see what simplification the function is making. Passing `1` simplifies normally. `2` factors out any numbers or variables with the constant attribute (set by `SetAttributes[{m, g}, Constant]`). `3` factors out numbers and constants, setting them to `1` at the end of the evaluation (the trivial 'go to 1' replacement rule `g21` is applied).

`g21` is a replacement rule, such that its semantic is `Ex/.g21[]`. 

---

### REFERENCE

**Legend:**
- \* Toggles with a comma or a `,0`.
- \*\* Optional parameters and transformation layers.
- \*\*\* Both

| Function | Description |
|----------|-------------|
| `CsQ` | ConstantQuery. Return the variables with the constant attribute within an expression. |
| `Flatn`** | Applies Flatten. `Flatn[vec,x]` also removes any instances of `x`'s in `vec`. |
| `FNCheck` | Compare a custom function to a built in one. |
| `Intg` | Returns the integrand from some expression. |
| `Intl` | Returns limits of integration. |
| `Mag` | Returns a magnitude. Naive polymorphism. |
| `RP`* | Replace. `RP[x,{x0,y0},{xf,yf}]` sets all instances of `x0` and `y0` to `xf` and `yf`. |
| `SS`** | Solve with a cleaner return format. Tab through multiple solutions with the third argument. |
| `S`*** | Simplify with varying configurations. |
| `g21`*** | Set constants and constant attributed variables to `1`. |
| `Trm` | Returns the vector of addends in an expression. |
| `Vars`** | Returns an expressions dependent variables. Uses differentiation to get rid of constants if the second input argument is `Null` or `0`. |
| `d` | Total derivative. |
| `PRM`** | Parametrize variables. Unparametrize (...if the expression isn't a total mess ...Integrals are hard) with `1` in the second input argument. |
