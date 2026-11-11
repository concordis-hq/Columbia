# Cheat sheet: section 3, complexity control with the lasso

## The whole section in three sentences

1. Jana fit 24 coefficients. I ask whether that is too many for our data.
2. I add a penalty on coefficient size (the lasso), derive the algorithm that solves it, and let held-out
   data choose how strong the penalty is.
3. Held-out data chooses almost no penalty, so the answer is no: our model is not overfit.

If you lose your place at any point, say sentence 3 and move on.

---

## What to delete

| Slide | Keep or delete | Why |
|---|---|---|
| 1. Section title | Keep | Matches the other sections. Say two sentences. |
| 2. Add a penalty | Keep | States the problem and defines eta. |
| 3. Derivation (1/2) | Keep, say little | Needed so the update is derived, not just stated. |
| 4. Derivation (2/2) | Keep | This is the heart of the nugget. |
| 5. Pseudocode 1 | Keep | The nugget he assigned you. |
| 6. Pseudocode 2 | Keep | The part he described in most detail. |
| 7. Demo figure | Keep | Lets you show the result without switching to the notebook. |
| 8. Result | Keep | The takeaway is already on it in bold. |
| 9. What it tells us | **Delete** | Say its one point aloud over slide 8. |

That leaves 8 slides. Do not run the notebook live unless he asks; have it open in a browser tab, already
run, so you can show it if he does.

---

## The ten concepts, in the order you need them

**1. Overfitting.**
A model with too many coefficients starts fitting noise in the data it was trained on, and then predicts
new data worse. The question for our model: is 24 coefficients too many?

**2. The penalty and eta.**
Add a cost to the sum of squares: eta times the total size of the coefficients. Eta is a dial.
Eta = 0 is plain OLS. Turning eta up forces coefficients toward zero and makes the model simpler.

**3. Ridge versus lasso.**
Two ways to measure "size". Ridge squares the coefficients (p = 2); lasso takes absolute values (p = 1).
Ridge has a formula and only shrinks. Lasso has no formula, but sets some coefficients exactly to zero,
so it picks which features to keep.

**4. Why the lasso has no formula.**
The absolute value |b| is V-shaped, with a corner at 0. A corner has no slope, so "take the derivative and
set it to zero" does not work there. And 0 is exactly where the lasso wants to put coefficients.

**5. The trick: one coefficient at a time.**
Hold 23 coefficients fixed and look at just one. Now the problem is a parabola plus a V, in one variable.
That can be solved by hand. This is called coordinate descent.

**6. Rho, the pull.**
For the one coefficient you are solving, rho measures how much its feature still has to explain after the
other features have done their share. Big rho: the feature matters. Small rho: it adds little.

**7. The soft-threshold.**
The solution of the one-variable problem. If the pull rho is weaker than eta, the coefficient is exactly 0.
If it is stronger, the coefficient is the pull reduced by eta. Example with eta = 1: a pull of 3 becomes 2,
a pull of 0.4 becomes 0.

**8. Cycling.**
Changing one coefficient changes the pull on the others, so you go round all 24 again and again until
nothing moves. That loop is Pseudocode 1.

**9. Cross-validation.**
How to choose eta without cheating. Hide one fifth of the data, fit on the rest, test on the hidden fifth.
Rotate so each fifth is hidden once. The eta with the lowest error on hidden data wins (eta star).
We hide whole artists, not single sales, because one artist's sales look alike.

**10. One standard error (eta_1se).**
The error estimate is noisy, so several values of eta are effectively tied with the best. Eta_1se is the
strongest penalty among those ties: the simplest model the data cannot tell apart from the best one.

---

## One sentence per slide (the minimum you must say)

1. "Jana fit 24 coefficients; I ask whether that is more than the data supports."
2. "We add a penalty, eta times the sum of absolute values. That is the lasso, and it has no closed form because the absolute value has a corner at zero."
3. "Hold every coefficient fixed except one, and the problem becomes a parabola plus an absolute value in one variable."
4. "Solving that by cases gives the soft-threshold: if a feature's pull is weaker than eta, its coefficient is exactly zero; otherwise shrink it by eta."
5. "The algorithm applies that update to each coefficient in turn and repeats until nothing changes."
6. "To choose eta, we hold out one fifth of the artists, fit on the rest, and pick the eta with the lowest held-out error."
7. "Held-out error falls as we relax the penalty, then goes flat; on the right, features enter one at a time, size first."
8. "The best eta is so small that the lasso is essentially OLS: 0.380 against 0.381 on 2026 sales. Our model is not overfit."

---

## Numbers to know

| Number | What it is |
|---|---|
| 21,783 | sales in the training set (2023 to 2025) |
| 8,458 | sales in the test set (2026) |
| 24 | coefficients, not counting the intercept |
| 5 | folds in cross-validation |
| 0.380 and 0.381 | R squared on 2026 sales: OLS, then lasso at the best eta |
| 0.0001 against 0.056 | gain in held-out error, against its standard error: noise |
| 4 dropped, 0.372 | what the stronger penalty (eta_1se) does: 20 of 24 kept, R squared 0.372 |
| about 900 | sales per coefficient, which is why there is no overfitting |
| 11 seconds | time for the notebook to run top to bottom |

---

## How to say the symbols

- **η** "AY-tuh" (eta): the penalty weight
- **β** "BAY-tuh" (beta): a coefficient
- **ρ** "row" (rho): the pull on one coefficient
- **β̂** "beta hat": an estimated coefficient
- **η\*** "eta star": the best eta
- **G** is X-transpose-X, **c** is X-transpose-y: the two pieces of Jana's normal equation

---

## Five questions most likely to come up

**"Why can't you just take the derivative?"**
The absolute value has a corner at zero, so it has no derivative there, and zero is where the solution
puts many coefficients.

**"Why does the lasso give exact zeros and ridge doesn't?"**
The corner. With ridge, the penalty is smooth and gets weaker and weaker near zero, so a coefficient gets
small but never reaches it. With the lasso, the penalty pushes just as hard at zero as anywhere else, so a
weak feature is pushed all the way.

**"So the lasso did nothing?"**
It answered the question. With about 900 sales per coefficient there is nothing to overfit, so the best
penalty is almost zero. That is evidence our 24-coefficient model is the right size.

**"Why was (ln A) squared dropped when it is so significant?"**
It moves almost in step with ln A, so the lasso keeps one of the pair. It is not saying the size curve is
a straight line.

**"Why split by artist?"**
An artist's sales share fame and exposure. If the same artist were in both the fitting data and the hidden
data, the model would look better than it is on a new artist.

---

## If something goes wrong

- **You forget a step in the derivation:** say "the three cases are on the slide" and read the boxed line.
- **He asks for a proof of convergence:** "I am citing Tseng 2001 for that; I derived the update, not the convergence."
- **He asks something you don't know:** "I don't know; I would check it in the notebook." Do not guess.
- **The live notebook is asked for and fails:** the saved outputs are in the file; scroll to them.

## Three things not to say

- Don't say the lasso "improved" the model. The gain is noise.
- Don't say (ln A)² "doesn't matter". It was dropped because of collinearity.
- Don't call eta "lambda". Anastasia's section uses lambda, and eta for something else.
