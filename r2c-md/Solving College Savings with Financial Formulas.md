---
created: 2025-07-19
---
# From Iteration to Insight: Solving College Savings with Financial Formulas

This document explores a common financial planning challenge: determining the monthly savings required to fund a college education. We will begin with a real-world problem, analyze an intuitive but inefficient numerical solution, and then transition to a highly efficient analytical solution derived from fundamental mathematical principles. This journey highlights the power of applying theoretical concepts to achieve both accuracy and performance in practical applications.
### 1. The Real-World Problem

The goal is to calculate the precise monthly contribution needed to save for a four-year college education.

**Key Constraints & Context:**
- **Total Cost:** A known total cost for a four-year degree.
- **Withdrawals:** The cost is paid in four equal installments, withdrawn at the **start** of each college year.
- **Investment Growth:** All savings are invested and grow at a fixed annual interest rate, compounded monthly.
- **Starting Point:** There may be an existing balance in the savings account.
- **The Goal:** The investment account should have a balance of exactly zero after the fourth and final college payment is made.

This is a classic financial planning problem that requires careful accounting for the time value of money—the principle that money today is worth more than the same amount in the future due to its potential to earn interest.

### 2. The Bisection Method: An Intuitive Numerical Solution

A logical first approach is to build a simulation. We can create a function that takes a `monthly_contribution` as input and simulates the account balance year by year through the savings and college periods. The goal is to find the input that results in a final balance of zero.

The **bisection method** is a numerical root-finding algorithm that works perfectly for this.

The Concept:

If you have a continuous function and can find two points where the function's output has opposite signs (one positive, one negative), you know a root (where the function equals zero) must exist between them. The algorithm repeatedly guesses the midpoint of the interval and narrows the search to the half where the root must lie.

**Application to the Problem:**

- **Function:** `simulate_balance(monthly_contribution)` which returns the final balance.
- **Goal:** Find the `monthly_contribution` where `simulate_balance` returns `0`.
- **Process:**
    1. Start with a low guess (e.g., `$0`) and a high guess for the monthly contribution.
    2. Calculate the final balance using the midpoint of the guess range.
    3. If the balance is too high, the midpoint becomes the new high guess.
    4. If the balance is too low, the midpoint becomes the new low guess.
    5. Repeat until the balance is acceptably close to zero.

Here is the Python code for this approach:

```
def college_schedule_bisection(
        college_cost: float,
        annual_rate: float,
        kid_current_age: int,
        start_balance: float = 0.0
):
    """
    Finds the required monthly contribution using a bisection root-finding method.
    NOTE: This code contains a subtle logic error in how it calculates the goal,
    but it correctly demonstrates the bisection algorithm.
    """
    # This inner function simulates the final balance for a given contribution
    def simulate_balance(monthly_contribution):
        # ... A full simulation of savings and withdrawals would go here ...
        # This is a complex process and the slow part of the algorithm.
        # For brevity, we'll represent it conceptually.
        # It would return the balance left after the 4th college withdrawal.
        return 0 # Placeholder for a complex simulation

    # Bisection method implementation
    lower_bound = 0.0
    # A generous upper bound, e.g., the total cost divided by number of months
    upper_bound = (college_cost / max(1, 18 - kid_current_age)) / 12 * 2
    
    for _ in range(100): # Iterate to converge on the answer
        mid_contribution = (lower_bound + upper_bound) / 2
        final_balance = simulate_balance(mid_contribution) # SLOW STEP

        if abs(final_balance) < 0.01:
            return mid_contribution
        elif final_balance > 0:
            upper_bound = mid_contribution
        else:
            lower_bound = mid_contribution
            
    return (lower_bound + upper_bound) / 2
```

**Analysis:** This method is robust and intuitive. However, it is computationally expensive as it requires running a full simulation (the `SLOW STEP`) potentially hundreds of times to find the answer.

#### Example Calculation (Bisection Method)

Let's use a concrete example:
- **College Cost:** `$200,000`
- **Annual Rate:** `6%`
- **Kid's Current Age:** `10` (8 years to save)
- **Start Balance:** `$25,000`

The algorithm would work like this:
1. **Set Bounds:** `lower_bound = $0`, `upper_bound = $4,167`.
2. **Iteration 1:**
    - Guess `mid_contribution = ($0 + $4167) / 2 = $2083.50`.
    - Run `simulate_balance($2083.50)`. The simulation calculates that this contribution is far too high, leaving a large positive balance at the end.
        
    - **Result:** The answer is between $0 and $2083.50. The new `upper_bound` is `$2083.50`.
        
3. **Iteration 2:**
    
    - Guess `mid_contribution = ($0 + $2083.50) / 2 = $1041.75`.
        
    - Run `simulate_balance($1041.75)`. The simulation finds this contribution is too low, resulting in a negative balance (debt).
        
    - **Result:** The answer is between $1041.75 and $2083.50. The new `lower_bound` is `$1041.75`.
        
4. **Iteration 3:**
    
    - Guess `mid_contribution = ($1041.75 + $2083.50) / 2 = $1562.63`.
        
    - Run `simulate_balance($1562.63)`. This is still too high (positive balance).
        
    - **Result:** The answer is between $1041.75 and $1562.63. The new `upper_bound` is `$1562.63`.
        

This process continues, halving the search interval with each step until it converges on the correct answer (which we'll find is ~$1,161).

### 3. The Formula Method: An Efficient Analytical Solution

Mathematics provides a more direct path. The problem can be solved using standard financial formulas derived from the principles of annuities. This approach eliminates all iteration.

**The Concept:**

- **Annuity:** A series of fixed payments over a set period. Our college withdrawals and monthly savings are both annuities.
    
- **Present Value of an Annuity Due:** This formula calculates the lump sum needed at the **start** of a period to fund a series of future withdrawals that also happen at the start of each sub-period (e.g., each year). This is perfect for calculating our total savings goal on the child's 18th birthday.
    
- **Future Value of an Annuity (Sinking Fund):** This formula calculates the fixed payment required to reach a specific future savings goal. This is perfect for determining the `monthly_contribution`.
    

**Application to the Problem:**

1. **Step 1: Find the Goal.** Use the "Present Value of an Annuity Due" formula to calculate the exact lump sum needed on the day college starts.
    
2. **Step 2: Find the Path.** Use the "Future Value of an Annuity" formula to calculate the monthly savings needed to grow from the `start_balance` to the goal amount from Step 1.
    

This replaces the entire bisection loop with two direct calculations.

```
def college_schedule_corrected(
        college_cost: float,
        annual_rate: float,
        kid_current_age: int,
        start_balance: float = 0.0
):
    """
    Calculates the required monthly savings contribution using direct analytical formulas.
    """
    annual_college_cost = college_cost / 4
    monthly_rate = annual_rate / 12
    n_months_saving = max(0, 18 - kid_current_age) * 12

    # Step 1: Calculate the lump sum needed at age 18 (PV of an Annuity Due)
    effective_annual_rate = (1 + monthly_rate) ** 12 - 1
    pv_factor = (1 - (1 + effective_annual_rate) ** -4) / effective_annual_rate
    required_at_college_start = annual_college_cost * pv_factor * (1 + effective_annual_rate)

    # Step 2: Account for existing savings and find the monthly contribution needed
    fv_existing = start_balance * ((1 + monthly_rate) ** n_months_saving)
    target_remaining = required_at_college_start - fv_existing

    if target_remaining <= 0:
        return 0.0

    # Sinking fund formula to find the required monthly payment
    numerator = target_remaining * monthly_rate
    denominator = (1 + monthly_rate) ** n_months_saving - 1
    monthly_contribution = numerator / denominator

    return monthly_contribution
```

**Analysis:** This solution is instantaneous and mathematically precise. It represents a significant improvement in efficiency by replacing an iterative search with a direct calculation.

#### Example Calculation (Formula Method)

Using the same numbers as before:

- **College Cost:** `$200,000`
    
- **Annual Rate:** `6%`
    
- **Kid's Current Age:** `10`
    
- **Start Balance:** `$25,000`
    

**Step 1: Calculate the Total Savings Goal**

- `annual_college_cost` = `$200,000 / 4` = `$50,000`
    
- `monthly_rate` = `0.06 / 12` = `0.005`
    
- `effective_annual_rate` = `(1 + 0.005)^12 - 1` = `6.16778%`
    
- `pv_factor` = `(1 - (1 + 0.0616778)^-4) / 0.0616778` = `3.4518`
    
- `required_at_college_start` = `$50,000 * 3.4518 * (1 + 0.0616778)` = **`$183,245.56`**
    
    - This is the total lump sum you need on the child's 18th birthday.
        

**Step 2: Calculate the Required Monthly Contribution**

- `n_months_saving` = `(18 - 10) * 12` = `96`
    
- `fv_existing` (Future Value of your start balance) = `$25,000 * (1 + 0.005)^96` = `$40,380.36`
    
- `target_remaining` (How much more you need to save) = `$183,245.56 - $40,380.36` = `$142,865.20`
    
- `numerator` = `$142,865.20 * 0.005` = `$714.326`
    
- `denominator` = `(1 + 0.005)^96 - 1` = `0.615214`
    
- `monthly_contribution` = `$714.326 / 0.615214` = **`$1,161.10`**
    

The formula gives us the precise answer instantly, without any iteration.

### 4. The Derivation: Connecting Theory to Practice

How do we get the magical-seeming formula from the previous section? It's derived from the fundamental, year-by-year calculation using the sum of a **finite geometric series**.

Let's derive the Present Value formula: `PV = P * ((1 - (1 + r)^-n) / r)`.

1. The Long Calculation: The Present Value (PV) is the sum of each payment (P) discounted back to today's value.
    
    PV=(1+r)1P​+(1+r)2P​+⋯+(1+r)nP​
2. Factor out the Payment (P):
    
    PV=P[(1+r)11​+(1+r)21​+⋯+(1+r)n1​]
3. **Identify the Geometric Series:** The terms in the brackets form a geometric series where:
    
    - First Term (a): a=frac11+r  
        
    - Common Ratio (q): q=frac11+r  
        
    - Number of Terms: n  
        
4. Apply the Geometric Series Sum Formula: The sum of a finite geometric series is Sum=afrac1−qn1−q. Substituting our values:
    
    Sum=1+r1​⋅1−1+r1​1−(1+r1​)n​
5. Simplify the Expression: The denominator simplifies to fracr1+r. The (1+r) terms cancel out.
    
    Sum=r1−(1+r1​)n​=r1−(1+r)−n​
6. Reintroduce the Payment (P):
    
    PV=P[r1−(1+r)−n​]

This derivation shows that the compact formula is not arbitrary; it is the elegant, simplified result of the long-form summation, made possible by a core mathematical theorem. This is the bridge that connects the practical problem to the theoretical solution.