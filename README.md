# XisABisrB
IBM - October 2024 - Challenge

https://research.ibm.com/haifa/ponderthis/challenges/October2024.html
https://research.ibm.com/haifa/ponderthis/solutions/October2024.html

The Problem Statement:

We are looking for a natural number X satisfying the following properties of its decimal representation:

All the digits {1,2,3,4,6,7,8,9} appear at least once, but 0 and 5 do not appear at all.
X can be written as a front part A and back part B, such that if X were a string, it could be written as A concatenated with B (X = A || B).
B is the product of the digits of X (P(X)).
X is a multiple of B (X = rB, where r is an integer).
B is a perfect square.

My Formulae for Solution:

B is the product of X's digits:

B = (1 * 2 * 3 * 4 * 6 * 7 * 8 * 9) * 1^a * 2^b * 3^c * 4^d * 6^e * 7^f * 8^g * 9^h

The (1 * ... * 9) term covers mandatory digit appearances, 
and a,b,c...,h are counts of additional appearances of each digit. 
Thus we have

B = 1^(1+a) * 2^( (1+b) + 2*(1+d) + (1+e) + 3*(1+g) ) * 3^( (1+c) + (1+e) + 2*(1+h) ) * 7^(1+f)
B = 1^(1+a) * 2^(7 + b + 2d + e + 3g) * 3^(4 + c + e + 2h) * 7^(1 + f)

For B to be a perfect square (Rule 5), all its prime exponents in its factorization must be even.

B = 2^(8+2k) * 3^(4+2l) * 7^(2+2m)

A's Structure:
The relation X = rB implies A * 10^(length of B) = (r-1)B. 
As B is not divisible by 5, 
(r-1) must be divisible by 5^(length of B). 
r is of the form 5Z + 1 (its last digit is 1 or 6).

A = A' * 3^4 * 7^2 * 3^(2l) * 7^(2m) 
where A' is an integer not divisible by 5.

Input Parameters and Constraints: 

The search uses integer parameters a, b, c, d, e, g, h, m (ranged up to 10), which determine k, l, f:

k = (b + 2d + e + 3g - 1) / 2 (Constraint: b + 2d + e + 3g - 1 non-negative and even)
l = (c + e) / 2 + h (Constraint: c + e even)
f = 2m + 1 (Constraint: b + c + 3g odd)
A' is searched up to 152617 (excluding multiples of 5).

7 + b + 2d + e + 3g = 8 + 2k => k = (b + 2d + e + 3g - 1) / 2
4 + c + e + 2h = 4 + 2l => l = (c + e + 2h) / 2
1 + f = 2 + 2m => f = 1 + 2m 

Analysis of X = 8419411123272236924928:
The X value 8419411123272236924928 (with A = 84194111232, B = 72236924928) does not satisfy all conditions. 
Its B component (72236924928) is not a perfect square (~268769.29), violating Rule 5.

Python Code for Solution:
import math
import collections
import time

def is_perfect_square(n):
    """Checks if a number is a perfect square."""
    if n < 0:
        return False
    if n == 0:
        return True
    sqrt_n = int(math.isqrt(n))
    return sqrt_n * sqrt_n == n

def get_digits(n_val):
    """Returns a list of digits for a given number."""
    if n_val == 0:
        return [0]
    return [int(d) for d in str(n_val)]


def has_forbidden_digits(num_str):
    """Checks if a number string contains '0' or '5'."""
    return '0' in num_str or '5' in num_str

def solve_constrained_puzzle():
    start_time = time.time()

    # --- Defined Bounds for Variables ---
    # PARAMETER RANGES: Now extended to go up to 10 for all relevant parameters
    RANGE_ABCDEGH = range(0, 11)  # 0 to 10 inclusive, as per user's request
    RANGE_M = range(0, 11)       # 0 to 10 inclusive (already was)
    RANGE_APrime = range(1, 152617) # This will be manually updated by the user to 200001

    required_digits_set = {1, 2, 3, 4, 6, 7, 8, 9}

    print("Starting search for the smallest 3 X values with the following bounds:")
    print(f"a, b, c, d, e, g, h: {list(RANGE_ABCDEGH)}")
    print(f"m: {list(RANGE_M)}")
    print(f"A': {list(RANGE_APrime)[:5]} ... {list(RANGE_APrime)[-5:]} (excluding multiples of 5)")
    print(f"\n!!! WARNING: The parameter ranges (a-h) have been significantly expanded (up to 10). This search will be VERY time-consuming. !!!")
    print("Please be patient, as it may take a very long time to complete.")

    # Store top N solutions
    N_SOLUTIONS = 3
    found_solutions = [] # List to store dictionaries of solution details (for distinct X values)
    found_x_values = set() # To keep track of X values already stored to ensure distinctness

    # Store the maximum X value currently in found_solutions for quick comparison
    max_X_in_top_N = float('inf')


    # Outer loops iterate through the variables that directly define A and B
    for a in RANGE_ABCDEGH:
        for b in RANGE_ABCDEGH:
            for c in RANGE_ABCDEGH:
                # Early pruning: c+e must be even (for l)
                for e in RANGE_ABCDEGH:
                    if (c + e) % 2 != 0:
                        continue
                    for d in RANGE_ABCDEGH:
                        for g in RANGE_ABCDEGH:
                            # Early pruning: b+c+3g must be odd
                            if (b + c + 3 * g) % 2 == 0:
                                continue
                            # Early pruning: b+2d+e+3g-1 must be non-negative and even (for k)
                            k_numerator = b + 2 * d + e + 3 * g - 1
                            if k_numerator < 0 or k_numerator % 2 != 0:
                                continue
                           
                            k = k_numerator // 2 # Calculate k

                            for h in RANGE_ABCDEGH:
                                # Calculate l
                                l = (c + e) // 2 + h
                                if l < 0:
                                    continue

                                for m in RANGE_M:
                                    # Calculate f (number of 7s related to m)
                                    f = 2 * m + 1
                                   
                                    # --- Construct B_val first ---
                                    # B = 2^(8+2k) * 3^(4+2l) * 7^(2+2m)
                                    try:
                                        B_val = (pow(2, 8 + 2 * k) *
                                                         pow(3, 4 + 2 * l) *
                                                         pow(7, 2 + 2 * m))
                                    except OverflowError:
                                        continue

                                    B_str = str(B_val)
                                    if has_forbidden_digits(B_str):
                                        continue

                                    # --- Construct A_val ---
                                    for A_prime in RANGE_APrime:
                                        if A_prime % 5 == 0:
                                            continue
                                       
                                        try:
                                            A_val = (A_prime *
                                                                 pow(3, 4) *
                                                                 pow(7, 2) *
                                                                 pow(3, 2 * l) *
                                                                 pow(7, 2 * m))
                                        except OverflowError:
                                            continue

                                        A_str = str(A_val)
                                        if has_forbidden_digits(A_str):
                                            continue
                                       
                                        # --- Construct X = A || B and check rules ---
                                        X_str = A_str + B_str
                                       
                                        # Optimization: If X_str is already longer than the largest in top N, skip.
                                        if len(found_solutions) == N_SOLUTIONS and len(X_str) > len(str(max_X_in_top_N)):
                                            continue
                                       
                                        X_val = int(X_str)

                                        # Optimization: If current X_val is already larger than largest in top N, skip
                                        if len(found_solutions) == N_SOLUTIONS and X_val >= max_X_in_top_N:
                                            continue

                                        # NEW: Check if this distinct X_val has already been found
                                        if X_val in found_x_values:
                                            continue

                                        if has_forbidden_digits(X_str):
                                            continue

                                        digits_in_X_set = set(int(d) for d in X_str)
                                        if not required_digits_set.issubset(digits_in_X_set):
                                            continue
                                       
                                        product_of_digits_X = 1
                                        for char_digit in X_str:
                                            product_of_digits_X *= int(char_digit)
                                       
                                        if product_of_digits_X != B_val:
                                            continue
                                       
                                        if X_val % B_val != 0:
                                            continue
                                       
                                        # --- All rules satisfied! This is a valid solution. ---
                                        r_val = X_val // B_val
                                       
                                        current_solution = {
                                                    "X": X_val,
                                                    "A": A_val,
                                                    "B": B_val,
                                                    "A_prime": A_prime,
                                                    "a": a, "b": b, "c": c, "d": d, "e": e, "g": g, "h": h, "m": m,
                                                    "k": k, "l": l, "f": f,
                                                    "r": r_val,
                                                    "time_found": time.time() - start_time
                                                }

                                        if len(found_solutions) < N_SOLUTIONS:
                                            found_solutions.append(current_solution)
                                            found_x_values.add(X_val) # Add to distinct set
                                            found_solutions.sort(key=lambda sol: sol["X"])
                                            max_X_in_top_N = found_solutions[-1]["X"] if found_solutions else float('inf')
                                            print(f"\n--- Candidate X found: {X_val} (Currently in top {len(found_solutions)} distinct solutions). Time: {current_solution['time_found']:.2f}s ---")
                                        elif X_val < max_X_in_top_N:
                                            # Remove the largest X_val from the list and its value from the distinct set
                                            popped_solution = found_solutions.pop() # Assumes it's already sorted and largest is at end
                                            found_x_values.remove(popped_solution["X"])

                                            found_solutions.append(current_solution)
                                            found_x_values.add(X_val)
                                            found_solutions.sort(key=lambda sol: sol["X"])
                                            max_X_in_top_N = found_solutions[-1]["X"]


            # --- Final Output ---
            total_search_duration = time.time() - start_time
            if found_solutions:
                print("\n" + "="*40)
                print(f"     SMALLEST {N_SOLUTIONS} DISTINCT SOLUTIONS FOUND     ") # Updated wording
                print("="*40)
               
                for i, sol in enumerate(found_solutions):
                    print(f"\n--- Solution #{i+1} ---")
                    print(f"X = {sol['X']}")
                    print(f"A = {sol['A']}")
                    print(f"B = {sol['B']}")
                    print(f"r (X / B) = {sol['r']}")
                    print(f"Time found: {sol['time_found']:.2f} seconds")
                    print("Parameters that generated this solution:")
                    print(f"  A' = {sol['A_prime']}")
                    print(f"  a = {sol['a']}, b = {sol['b']}, c = {sol['c']}, d = {sol['d']}, e = {sol['e']}, g = {sol['g']}, h = {sol['h']}, m = {sol['m']}")
                    print(f"  Derived values: k = {sol['k']}, l = {sol['l']}, f = {sol['f']}")
                   
                print("\n" + "="*40)
                print(f"Total search time: {total_search_duration:.2f} seconds.")

            else:
                print("\n" + "="*40)
                print("  NO SOLUTION FOUND WITHIN BOUNDS  ")
                print("="*40)
                print(f"Total search time: {total_search_duration:.2f} seconds.")
                print("Consider expanding the search bounds for variables (a,b,c,d,e,g,h,m, A').")

# Execute the search
try:
    print("Attempting to call solve_constrained_puzzle()...")
    solve_constrained_puzzle()
except Exception as e:
    print(f"\nAn error occurred: {e}")
    import traceback
    traceback.print_exc()

Conclusion and Results:
After running the provided Python script with the specified parameter ranges (a,b,c,d,e,g,h,m from 0 to 10, and A' from 1 to 152616 excluding multiples of 5), the search successfully completed.

The three smallest distinct natural numbers X satisfying all the given properties are:

X = 1817198712146313216

(Found at approximately 0.97 seconds into the search)
Parameters that generated this solution: A' = 50872, a=0, b=0, c=0, d=1, e=0, g=1, h=1, m=0
Derived values: k=2, l=1, f=1
X = 119336716884276412416

(Found at approximately 1.19 seconds into the search)
Parameters that generated this solution: A' = 3712, a=0, b=0, c=0, d=1, e=0, g=3, h=2, m=0
Derived values: k=5, l=2, f=1
X = 131771911321316818944

(Found at approximately 1.09 seconds into the search)
Parameters that generated this solution: A' = 40988, a=0, b=0, c=0, d=1, e=0, g=1, h=2, m=0
Derived values: k=2, l=2, f=1
The total search time for these solutions within the defined bounds was approximately 67.88 seconds.

Best regards,

code provided by Gemini after many problems - seems to apply the math though.
