# Section 3 speaking script: complexity control with the lasso

About 9 to 10 minutes at a normal pace. Lines marked **[cut if short]** can go if your slot is tighter.
Numbers in the script are the ones the notebook prints.

---

## Slide 1. Section title (20 seconds)

Jana just fit our hedonic model by ordinary least squares: 24 slopes and an intercept, on 21,783 sales.

My question is the one that comes next. The world hands us data, not models. Are 24 coefficients more than
this data can support? I answer that with a penalty on the coefficients, and I let held-out data decide how
strong the penalty should be.

The method is the lasso, from Tibshirani in 1996, and the algorithm is coordinate descent, from Friedman,
Hastie and Tibshirani in 2010.

## Slide 2. Is the model too complex? Add a penalty (75 seconds)

We keep Jana's sum of squares and add one term: eta times the sum of the absolute values of the
coefficients, each raised to a power p.

Eta is the price the model pays for coefficient size. Eta equal to zero is OLS. A large eta pushes every
coefficient toward zero. On my slides eta always means this penalty weight.

There are two standard choices of p.

With p equal to 2, called ridge, the problem still has a closed form. It is Jana's normal equation with
2 eta times the identity added to X transpose X. It shrinks every coefficient but never sets one to zero.

With p equal to 1, the lasso, the absolute value has no derivative at zero. So we cannot differentiate, set
the gradient to zero and solve. There is no closed form, and we need an algorithm. In return, the lasso sets
some coefficients exactly to zero, so it selects features. That is the one I present.

My derivation has a start, an end and a tool. The start is the OLS sum of squares. The end is an update rule
for one coefficient. The tool is to minimize over one coordinate at a time, because that one-dimensional
problem we can solve by hand.

## Slide 3. Derivation (1/2): one coefficient at a time (75 seconds)

Step one. I standardize each column of X to mean zero and variance one, and I center y. Two reasons. The
intercept drops out, so we never penalize it. And one eta is then fair to every feature, whatever its units.

Step two. Expand the sum of squares, the same expansion Jana did. I write G for X transpose X and c for
X transpose y. Those are the two quantities in the normal equation.

Step three. Freeze every coefficient except beta j, and collect the terms that contain beta j. From the
quadratic term I get one half G j j beta j squared, plus beta j times the sum over the other columns of
G j k beta k. From the linear term I get minus c j beta j. Everything else is a constant.

So the objective, as a function of beta j alone, is a parabola, minus rho j times beta j, plus eta times the
absolute value of beta j.

Rho j is c j minus the sum over the other columns of G j k beta k. In words: how strongly column j lines up
with what the other columns have left unexplained.

## Slide 4. Derivation (2/2): the soft-threshold (90 seconds)

Now minimize that one-dimensional function. It is convex, and it has a kink at zero, so I take three cases.

If beta j is positive, the absolute value is just beta j, and I can differentiate. Setting the derivative to
zero gives beta j equals rho j minus eta, over G j j. That is positive only when rho j is bigger than eta.

If beta j is negative, the same steps give rho j plus eta, over G j j. That is negative only when rho j is
below minus eta.

If rho j is between minus eta and eta, neither case is consistent. Just left of zero the slope is negative
or zero, and just right of zero it is positive or zero. So the minimum sits exactly on the kink: beta j is zero.

All three cases fit in one line, the boxed update. S is called the soft-threshold. The plot shows it: flat
at zero in the shaded band, and outside the band it is the OLS line shifted toward zero by eta.

That flat band is the whole point. A feature must explain more than eta before it is allowed into the model.

One check. With eta equal to zero, the update is beta j equals rho j over G j j. That is Gauss–Seidel
iteration on the normal equation. So with no penalty, the algorithm returns Jana's OLS.

**[cut if short]** Each update can only lower the objective, and the objective is convex with a penalty that
separates across coordinates. That is the condition under which cycling converges to the lasso solution; the
proof is Tseng's, from 2001, and I am quoting it, not proving it.

## Slide 5. Pseudocode 1: lasso by coordinate descent (60 seconds)

Here is the algorithm. The inputs are standardized X, centered y, and eta.

Compute G and c once. Start with every coefficient at zero, so every feature is switched off.

Then sweep over the 24 coefficients. For each one, compute rho j, apply the soft-threshold, and divide by
G j j. Those two lines are exactly the derivation.

Repeat until the largest change in any coefficient is below ten to the minus ten.

The code in the notebook is these lines in NumPy. I checked it two ways. With eta equal to zero it reproduces
OLS. With eta above zero it agrees with scikit-learn's lasso to 8 decimal places.

## Slide 6. Pseudocode 2: choosing eta on held-out data (75 seconds)

The lasso gives a different model for every eta. This second algorithm picks one.

Split the data into five folds. For each fold, hold it out, fit on the other four, and measure the squared
error on the held-out fold. Do that for every eta on a grid, from the largest down to zero.

The grid starts at eta max, the largest absolute entry of c. At that value every coefficient is zero, which
you can read off the soft-threshold: starting from zero, every rho j equals c j, and none is outside the band.

Average the error over the five folds. Eta star is the value with the lowest held-out error.

I also report eta one-s-e: the largest eta whose error is within one standard error of the best. It is the
simplest model that held-out data cannot tell apart from the best one.

Two details matter. The folds are by artist, not by sale. Sales by the same artist share fame and exposure,
so splitting an artist across folds would let the model see the held-out artist. And the standardization
uses only the training folds, so the held-out fold never touches the fit.

## Slide 7. Demo: sweeping eta (60 seconds, plus the notebook run)

*Run the notebook here. It takes about 11 seconds and reads the data straight from our GitHub repository.*

On the left is held-out error against eta. Read it from left to right: the penalty gets weaker and the model
gets more complex. The error falls steeply, then goes flat. The red line is eta star and the orange line is
eta one-s-e.

On the right, each line is one coefficient. At the far left they are all zero. As eta shrinks they switch on
one at a time. The first to enter is size, log area, the green line at the bottom. Medium and the artist's
fame follow.

**[cut if short]** The two lines that swing apart on the right are log area and its square. I come back to
those on the last slide.

## Slide 8. Result: beta at eta star against beta at zero (75 seconds)

This is the comparison the method is for: the coefficients at the best eta, next to the coefficients at zero.

Cross-validation picks eta star so small that the lasso keeps all 24 coefficients, and they barely move.
Established artists: 0.837 under OLS, 0.835 under the lasso. Fame: 0.133 in both.

The gain in held-out error is 0.0001, against a standard error of 0.056. That is noise. On the 2026 sales,
which were never used to fit or to choose eta, R squared is 0.380 for OLS and 0.381 for the lasso.

If I push the penalty as far as the data allows, to eta one-s-e, four coefficients go to zero and R squared
on 2026 falls to 0.372. Three of the four are ones Jana's table already marked as weak: panel support, work
made after 2000, and work made before 1800.

## Slide 9. What complexity control tells us (60 seconds)

The main finding is that our hedonic model is not overfit. We have about 900 sales per coefficient, and
held-out error is flat all the way down to eta equal to zero. The 24 slopes are supported by the data.

Two limitations.

First, log area and its square are almost collinear. The lasso keeps one and drops the other. That says
something about the pair. It is not evidence that the size curve is a straight line, and Jana's Wald test
shows size is our strongest block.

Second, lasso coefficients are shrunk toward zero by construction. Read them for which features survive.
For how large an effect is, quote Jana's OLS table.

So the linear model is not too complex. The next question is whether it is too simple, and Amy tests that
with a random forest in section 5. First, Anastasia turns the question around and asks what the artwork
says about the auction house.

---

# Questions you should be ready for

**Why is there a one half in front of the sum of squares?**
Convenience. It cancels the 2 when you differentiate. Without it, the threshold would be eta over 2.

**Why not just use gradient descent?**
The absolute value has no gradient at zero, which is exactly where the lasso solution puts many
coefficients. Coordinate descent avoids the problem because each one-dimensional step has an exact answer.

**How do you know coordinate descent converges when the function is not differentiable?**
The objective is convex, and the non-smooth part is a sum of terms that each involve one coordinate.
Tseng (2001) proves convergence under that condition. In the notebook I also check the optimality
conditions directly: |x_j' r| is at most eta for every column, with equality where beta_j is nonzero.

**Why does eta = 0 take so many sweeps (1,585) when eta = 0.3 eta_max takes 6?**
ln A and (ln A)² are nearly collinear, so Gauss–Seidel zig-zags between them. A larger eta removes one of
the pair and the problem becomes easy.

**Why split the folds by artist?**
An artist's sales share fame and exposure. With an artist on both sides of the split, held-out error would
look better than it is for a new artist.

**Why is eta star so small? Does that mean the lasso was pointless?**
No. It answers the question. With 21,783 sales and 24 coefficients there is little variance to remove, so
the penalty has nothing to buy. It would matter with many more features, for example one dummy per artist:
there are 8,372 artists in the training set.

**Why did the lasso drop (ln A)² when its p-value is below 0.001?**
A p-value asks whether a coefficient is zero given all the others. The lasso asks which features are worth
their price for prediction. ln A alone carries most of what the pair predicts, so at a strong penalty the
second one is not worth keeping. The cost shows up as the drop in R squared from 0.380 to 0.372.

**What would ridge have given?**
A closed form, (X'X + 2 eta I)^-1 X'y, and no zeros. Cross-validation would work the same way.

**What do the coefficients mean after standardizing?**
The algorithm works on standardized columns. The table on slide 8 converts back to original units, so the
numbers sit next to Jana's.
