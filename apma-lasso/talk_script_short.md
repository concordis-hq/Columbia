# Section 3 script

## Slide 1. Title

Jana just fit our model with 24 coefficients.

My question: is 24 more than our data can support?

I test that with a method called the lasso.

## Slide 2. What the lasso is

We keep Jana's sum of squared errors and add a penalty: eta times the total size of the coefficients.

Eta is a dial. At zero we get OLS. Turning it up pushes coefficients toward zero.

With p equal to 2, ridge, there is a formula. With p equal to 1, the lasso, there is none, because the absolute value has a corner at zero.

So we need an algorithm. In return, the lasso sets weak coefficients exactly to zero.

## Slide 3. Derivation, part 1

We cannot solve for all 24 coefficients at once, so we solve for one and hold the others fixed.

After standardizing and expanding the sum of squares, the cost for one coefficient has three parts: a parabola, a term with rho, and the penalty.

Rho measures how useful that feature still is, after the other features have explained what they can.

## Slide 4. Derivation, part 2

Now we find the lowest point of that cost. We compare rho, how useful the feature is, with eta, the penalty.

If rho is bigger than eta, the coefficient is rho minus eta, divided by G.

If rho is below minus eta, it is rho plus eta, divided by G.

If rho is in between, the coefficient is exactly zero.

One rule covers all three cases. It is called the soft-threshold, and it is why the lasso drops weak features.

With eta equal to zero, it gives OLS back.

## Slide 5. Pseudocode 1

This is the algorithm. Start with every coefficient at zero.

For each coefficient, compute rho and apply the soft-threshold.

Repeat until nothing changes.

I checked it two ways: with no penalty it matches Jana's OLS, and with a penalty it matches scikit-learn.

## Slide 6. Pseudocode 2

That gives a model for every eta, so we need to choose one.

We split the artists into five groups. We hide one group, fit on the other four, and measure the error on the hidden group.

We do that for every eta and every group, then average.

Eta star is the penalty with the lowest error on hidden data.

We split by artist because one artist's sales look alike.

## Slide 7. The figure

On the left is the error on hidden data for each penalty. As the penalty gets weaker, the error drops and then goes flat.

If the model were overfitting, the curve would rise again. It does not.

On the right, each line is one coefficient. They switch on one at a time. Size comes first, then medium and fame.

## Slide 8. The result

At the best penalty, the lasso keeps all 24 coefficients, and they barely change from OLS.

On 2026 sales, R squared is 0.380 for OLS and 0.381 for the lasso. That difference is noise.

With a stronger penalty, four coefficients drop out and R squared falls to 0.372.

So the data chooses almost no penalty. Our model is not overfit.

## Slide 9. Takeaway (optional)

The main finding: our 24-coefficient model is the right size for this data.

Two limits. Log area and its square move together, so the lasso keeps only one. And lasso coefficients are shrunk, so for effect sizes use Jana's table.

Next, Anastasia asks what the artwork tells us about the auction house.
