
Using **Julia** and **R** to solve an example linear programming problem from the Coursera *Modeling Risks and Realities* course.

### Problem
ABC company builds widgets, and currently produces two versions: simple widgets and enhanced widgets. They are looking at the increase in sales of widgets through marketing efforts in two new markets: India and China.

The relative increase on net sales for each marketing dollar per version per market is as follows:

Version | India | China
--- | --- | ---
Standard | 0.05 | 0.04
Enhanced | 0.02 | 0.03

The marketing spend for the combined India and China markets is not to exceed \$195 million. The net increase for the India market across both product versions must be at least \$3 million, and the net increase for China across both versions must be at least \$4 million. The enhanced product version sales net increase must be at least 80\% of the net increase in the standard version sales.

### Decision Variables
There are two versions and two markets for each version, yielding four decision variables.

$ A_{si} = $ Amount spent on Standard version in India  
$ A_{sc} = $ Amount spent on Standard version in China  
$ A_{ei} = $ Amount spent on Enhanced version in India  
$ A_{ec} = $ Amonut spent on Enhanced version in China

### Objective Function
ABC company seeks to maximize net sales increase from both versions in both markets. The objective function can therefore be written:
$$ \max 0.05*A_{si} + 0.04*A_{sc} + 0.02*A_{ei} + 0.03*A_{ec} $$

### Constraints
Total spending: $A_{si} + A_{sc} + A_{ei} + A_{ec} \geq 195 $  
India requirement: $A_{si} + A_{ei} \geq 3 $  
China requirement: $A_{sc} + A_{ec} \geq 4 $  
Enhanced percentage requirement: $(0.02*A_{ei} + 0.03*A_{ec}) / (0.05*A_{si} + 0.04*A_{sc}) \geq 0.08 $

## Julia (JuMP)


```julia
using JuMP, Gurobi
```


```julia
m = Model(solver=GurobiSolver())
```




$$ \begin{alignat*}{1}\min\quad & 0\\
\text{Subject to} \quad\end{alignat*}
 $$




```julia
@variable(m, A_si >= 0) # Amount for standard model in India
@variable(m, A_sc >= 0) # standard model in China
@variable(m, A_ei >= 0) # enhanced model in India
@variable(m, A_ec >= 0) # enhanced model in China
```




$$ A_ec $$




```julia
@objective(m, Max, 0.05A_si + 0.04A_sc + 0.02A_ei + 0.03A_ec)
```




$$ 0.05 A_si + 0.04 A_sc + 0.02 A_ei + 0.03 A_ec $$




```julia
@constraint(m, total_spending, A_si + A_sc + A_ei + A_ec <= 195) #in millions
@constraint(m, india, 0.05A_si + 0.02A_ei >= 3) # net increase in India is at least 3 million
@constraint(m, china, 0.04A_sc + 0.03A_ec >= 4) # net increase in China is at least 4 million
@constraint(m, enhanced, (0.02A_ei + 0.03A_ec) >= 0.8(0.05A_si + 0.04A_sc)) # enhanced model is at least 80% increase of standard model
```




$$ 0.02 A_ei + 0.03 A_ec - 0.04000000000000001 A_si - 0.032 A_sc \geq 0 $$




```julia
print(m) # show model formulation
```

    Max 0.05 A_si + 0.04 A_sc + 0.02 A_ei + 0.03 A_ec
    Subject to
     A_si + A_sc + A_ei + A_ec ≤ 195
     0.05 A_si + 0.02 A_ei ≥ 3
     0.04 A_sc + 0.03 A_ec ≥ 4
     0.02 A_ei + 0.03 A_ec - 0.04000000000000001 A_si - 0.032 A_sc ≥ 0
     A_si ≥ 0
     A_sc ≥ 0
     A_ei ≥ 0
     A_ec ≥ 0



```julia
solve(m)
```

    Optimize a model with 4 rows, 4 columns and 12 nonzeros
    Coefficient statistics:
      Matrix range    [2e-02, 1e+00]
      Objective range [2e-02, 5e-02]
      Bounds range    [0e+00, 0e+00]
      RHS range       [3e+00, 2e+02]
    Presolve time: 0.01s
    Presolved: 4 rows, 4 columns, 12 nonzeros
    
    Iteration    Objective       Primal Inf.    Dual Inf.      Time
           0    1.4000000e+29   4.352000e+30   1.400000e-01      0s
           3    7.3828125e+00   0.000000e+00   0.000000e+00      0s
    
    Solved in 3 iterations and 0.03 seconds
    Optimal objective  7.382812500e+00





    :Optimal




```julia
getobjectivevalue(m)
```




    7.3828125



## RCall (R in Julia)


```julia
using RCall
```


```julia
R"""
library(lpSolveAPI)
library(magrittr)
m2 <- make.lp(0,4)
lp.control(m2, sense="max")
set.objfn(m2, obj = c(0.05, 0.04, 0.02, 0.03))
row.add.mode(m2, "on")
add.constraint(m2, xt = c(1, 1, 1, 1), type = "<=", rhs = 195, indices = c(1:4))
add.constraint(m2, xt = c(0.05, 0.02), type = ">=", rhs = 3, indices = c(1,3))
add.constraint(m2, xt = c(0.04, 0.03), type = ">=", rhs = 4, indices = c(2,4))
add.constraint(m2, xt = c(-0.04, -0.032, 0.02, 0.03), type = ">=", rhs = 0, indices = c(1:4))
row.add.mode(m2, "off")
set.bounds(m2, lower = c(0,0,0,0), upper = rep(195, 4))
"""
```




    RCall.RObject{RCall.NilSxp}
    NULL





```julia
R"""
solve(m2)
get.variables(m2) %>% divide_by(sum(get.variables(m2))) %>% round(3)
vars <- get.variables(m2)
"""
@rget vars
```




    4-element Array{Float64,1}:
      67.6562
      17.9688
       0.0   
     109.375 




```julia
R"""
outcome <- get.objective(m2)
"""
@rget outcome
```




    7.3828125



Solutions:

| Objective | $A_{si}$ | $A_{sc}$ | $A_{ei}$ | $A_{ec}$
--- | --- | --- | --- | --- | ---
Julia | {{getobjectivevalue(m)}} | {{getvalue(A_si)}} | {{getvalue(A_sc)}} | {{getvalue(A_ei)}} | {{getvalue(A_ec)}}
R | {{print(outcome)}} | {{print(vars[1])}} | {{print(vars[2])}} | {{print(vars[3])}} | {{print(vars[4])}}

## Conclusion
Both Julia's JuMP package and R's lpSolveAPI package (accessed through Julia's RCall package) produced the same results, which is cool. Also, the RCall package allows for blending Julia and R inside of the Jupyter notebook, which is also pretty cool. So, in summary, cool. Cool cool cool.
