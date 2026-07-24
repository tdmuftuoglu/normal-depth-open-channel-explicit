# Normal depth in trapezoidal open channels

Reproduction code for the paper *Normal depth in trapezoidal open channels: an
explicit solution with certified error bounds*.

A single self-contained script regenerates every certified supremum, every table
entry and every figure in the paper from scratch. No input data are required and
no configuration is needed.

---

## The problem

For a trapezoidal channel, Manning's formula gives the discharge from the depth
in closed form, but not the depth from the discharge. A large literature of
explicit approximations exists to avoid the resulting iteration, and each is
advertised by a single number: a maximum relative error over a stated range.

That number is almost never accompanied by the procedure that produced it. Where
sampling is described, the maximum is taken over a finite set of points, and a
maximum over a finite sample is a lower bound on the true supremum rather than
the supremum itself.

This code does three things:

1. **Certifies** published accuracy claims by adaptive supremum search rather
   than fixed-grid sampling.
2. **Separates** the two design levers available when a fixed-point iteration is
   truncated into an explicit formula — how deep the iteration is unrolled, and
   how many of its constants are released to an optimiser — and measures each
   with the other held fixed.
3. **Proposes** a new explicit solution built on the measured ranking.

---

## Running it

### Google Colab

```python
!wget -q https://raw.githubusercontent.com/USER/REPO/main/reproduce_trapezoidal_normal_depth.py
!python reproduce_trapezoidal_normal_depth.py
```

Replace `USER/REPO` with this repository's path. Nothing else is needed; Colab
ships with all three dependencies.

### Locally

```bash
python reproduce_trapezoidal_normal_depth.py
```

Requires NumPy, SciPy and Matplotlib. No other dependencies, no installation
step, no arguments.

### Runtime

Roughly sixteen minutes on one CPU core. Progress is printed continuously and
every step reports elapsed time, so a run that appears to have stalled can be
told apart from one that is still working.

All random seeds are fixed. Repeated executions on the same platform return
identical output.

---

## What it writes

| File | Contents |
|---|---|
| `results_log.txt` | The complete numerical session, exactly as printed |
| `coefficients.txt` | The fitted coefficients, plus a JSON record of every reported quantity |
| `fig1_accuracy.png` / `.pdf` | Error curves, and accuracy against side slope |
| `fig2_certification.png` / `.pdf` | Error surface, and fixed-grid reliability |
| `fig3_structure.png` / `.pdf` | Unrolling depth against coefficient freedom, and extrapolation |

---

## What it reproduces

**Audit of published solutions.** Each is certified over the range its own
source claims, in the variable that source uses.

| Solution | Claimed | Certified |
|---|---|---|
| Vatankhah & Easa (2011), Eqs. (23)–(24) | 0.70 % | 0.693 % |
| Vatankhah (2013), Eqs. (29)–(30) | 0.25 % | 0.238 % |
| Vatankhah (2023), Eqs. (9)–(10) | 0.70 % | 0.695 % |
| Vatankhah (2023), Eqs. (12)–(13) | 0.72 % | 0.715 % |
| Elhakeem (2017), Eqs. (15)–(16) | ~14 % | 15.984 % |

Every stated claim is confirmed. This is a negative result and is reported as
one.

**The two levers, separated.**

| Configuration | Fitted coefficients | Certified supremum |
|---|---|---|
| Two applications, published seed | 6, in the seed | 0.2379 % |
| Two applications, map freed, seed held | 14, in the map | ≈0.17 % |
| Three applications, published seed unchanged | none refitted | 0.0525 % |

Releasing the map's constants buys a factor of 1.4. Applying the map once more,
with nothing refitted at all, buys a factor of 4.5. The improvement from further
applications is geometric: the derivative of the map is bounded by 0.242 over
the domain, so the error contracts by at least a factor of 4.1 per application,
and the observed ratios are 4.5, 4.3 and 4.3.

**The proposed solution.**

```
z*    = z/sqrt(1+z^2) - 1
eps   = 4*[ z(1+z^2)^(1/3) nQ / (lambda sqrt(S0) b^(8/3)) ]^(3/5)

t0    = -z* + [ 1.028077*z* + 1.058232*(1 + eps/(z*+1.894670)^0.311905)^0.475387 ]^1.328345
t     <- sqrt(1 + eps*(z* + t)^0.4)        applied three times

eta_n = (t - 1)/2 ,   y_n = eta_n*b/z
```

Certified supremum **0.0320 %** for z >= 0.25, using the same six coefficients
as the most accurate published alternative and one fewer than the next. The
error stays at 0.0320 % as the depth is taken down to 1e-8 and as the side slope
is taken up to 500, and remains below 0.17 % at a dimensionless depth of 1e6 —
six orders of magnitude beyond the range over which the coefficients were
fitted.

---

## A note on the objective

Fitting and reporting are deliberately kept apart. The coefficients are obtained
by minimising the maximum over a fixed 110 x 34 grid, because a global
optimisation calling the adaptive search at every function evaluation would be
prohibitive. The resulting expression is then certified by the adaptive
procedure, and the certified figure is the one reported.

The two are not the same. For the coefficients above, the fitting grid returns
0.0319 % while the certified supremum is 0.0320 %. The gap is small, but it is
of exactly the kind this study documents.

---

## Environment

Verified on:

```
python 3.12.13 | numpy 2.0.2 | scipy 1.16.3 | matplotlib 3.10.0
```

Older SciPy releases that do not accept `x0` in `differential_evolution` are
handled by a fallback path.

---

## Citation

If this code is used, please cite the paper and this archive:

```
Müftüoğlu, T.D. Normal depth in trapezoidal open channels:
an explicit solution with certified error bounds.

Archive: [Zenodo DOI to be inserted]
```

---

## Licence

[to be chosen — MIT and CC-BY-4.0 are both common for code accompanying a paper]
