---
title       : "Day 5. Model optimization"
author      : "Dominique Gravel"
job         : "Laboratoire d'écologie intégrative"
logo        : "logo.png"
framework   : io2012       # {io2012, html5slides, shower, dzslides, ...}
highlighter : highlight.js  # {highlight.js, prettify, highlight}
hitheme     : tomorrow      #
mode        : selfcontained
knit        : slidify::knit2slides
widgets     : [mathjax]
url:
  lib   : ./libraries
license     : by-nc-sa
assets      :
  css: "https://maxcdn.bootstrapcdn.com/font-awesome/4.6.0/css/font-awesome.min.css"

---

# Optimization 
## Finding equilibrium solution of a system of differential equations

<div style='text-align:center;'>
<img src="assets/img/RM_model.png" width="500px"></img>
</div>

---

# Optimization 

## Finding maximum likelihood estimates of a statistical model

<div style='text-align:center;'>
<img src="assets/img/regression.png" width="500px"></img>
</div>

---

# Optimization 

## Solution to complex management problems 

<div style='text-align:center;'>
<img src="assets/img/metapop.png" width="450px"></img>
</div>

---

# Objectives

Give an overview of the diversity of techniques and approaches

- Formulate a problem to maximize
- Find maximum values by simulated annealing

---

# A clasic from high scool maths

<div style='text-align:center;'>
<img src="assets/img/polynomial.png" width="750px"></img>
</div>

---

# Many algorithms

- Brute force/direct search
- Derivative-based methods
- Nelder-mean (simplex)
- Genetic algorithm
- Simulated annealing
- Monte Carlo Markov Chain
- Approximate Bayesian Computing

---

#  Which one to pick ? 

- The equation to optimize
- The size of the dataset
- Number of parameters
- Optimization surface
- Area for the optimal solution or precise value ?
- Single solution or a distribution of good solutions ?

---

# Cost function
## All methods do have in common a function to optimize
- The function takes parameters as arguments and return a single value to either maximize or minimize
- Parameters may be elements in an equation (e.g. regression coefficients) or any other condition we want to change (e.g. which habitat to protect)
- Value to optimize can be of any kind : R2, likelihood, residual error, abundance, occupancy, cost etc....
- All methods do have in common that they vary the input parameters and evaluate if the new parameters improve the result 
- Methods differ in i) the way they propose candidate parameters and ii) how they accept/reject propositions

---

# The problem to solve  

Find the value of X that minimizes the polynomial 

<div style='text-align:center;'>
$Y = 1 + 2X + 3X^2$
</div>

---

# The problem to solve  

For such a simple problem, we simply have to take the derivative and solve it for 

<div style='text-align:center;'>
$dY/dX = 0$
</div>

---

# The problem to solve  

## But imagine if the surface looks like  

<div style='text-align:center;'>
<img src="assets/img/surface.png" width="500px"></img>
</div>

---

# The problem to solve  

- Another solution may be to try bunch of values (random or regularly spaced) and pick the best. 
- The problem is that we'll never have the certainty that it's the best solution.
- May be very inefficient for problems with a lot of parameters

---

# Simulated annealing 

<div style='text-align:center;'>
<img src="assets/img/SA.jpeg" width="750px"></img>
</div>

---

# Simulated annealing

The name of the method is coming from metallurgy, by analogy to the movement of atoms in a hot piece of metal as it cools down. Simulated annealing interprets cooling as a slow decrease in the probability of accepting a bad proposition of parameter values as it explores the solution space. Accepting worse solutions is a fundamental property of the method which allows an extensive search of the solution space and avoid getting stuck on local optima. The method is an adaptation of the Metropolis–Hastings algorithm.

---

# Core principle 

Accept the candidate parameter value if it improves the solution

Accept a wrong candidate parameter value with probability

<div style='text-align:center;'>
	$\rho = exp(\Delta h/T_t)$
</div>

---

# Pseudo code

```
DEFINE Function to optimize h(X)
DEFINE Sampling function s(X)
DEFINE Temperature sequence T(steps)
```

---

# Pseudo code

```
REPEAT
    DRAW sample X1 from s(X0)
	EVALUATE function h(X1)
	COMPARE diff = h(X1) - h(X0)
	IF diff > 0 ACCEPT
	ELSE
		CALCULATE acceptance probability p = exp(diff_h\T)
		DRAW value P from random uniform on (0,1)
			IF P < p
			   accept X
			ELSE
		    	reject X
	UPDATE temperature using T(step)
UNTIL nsteps is reached
```

---

# Exercise 1

Find the value of X that minimizes the polynomial 

<div style='text-align:center;'>
$Y = 1 + 2X + 3X^2$
</div>

Tip : build a modular code with distinct functions so that it could easily be adapted to another problem 

---

# Exercise 2

## Imagine that you have to create a network of protected areas in a region where there is a metapopulation of an endangered frog. You have a limited amount of money to buy only 10 out of the 25 remaining forest fragments where the species could occur. Which one would you pick ? 

---

# Levins metapopulation model

## Create a random spatial network


```r
n = 25
r = 0.5
XY = cbind(runif(n), runif(n))
distMat = as.matrix(dist(XY, method = 'euclidean', upper = T, diag = T))
adjMat = matrix(0, nr = n, nc = n)
adjMat[distMat < r] = 1
diag(adjMat) = 0
```

---

# Levins metapopulation model

## Plot the spatial network


```r
dev.new(height = 11, width = 12)
par(mar=c(5,6,2,1))
plot(XY[,1],XY[,2],xlab = "X", ylab = "Y",cex.lab = 1.5, cex.axis = 1.25,cex = A, pch = 19)
adjVec = stack(as.data.frame(adjMat))[,1]
XX = expand.grid(XY[,1],XY[,1])
YY = expand.grid(XY[,2],XY[,2])
XX = subset(XX,adjVec==1)
YY = subset(YY,adjVec==1)
arrows(x0 = XX[,1],x1=XX[,2],y0 = YY[,1], y1 = YY[,2], length = 0,lwd = 0.1)
```

---

# Levins metapopulation model

## Plot the spatial network

<div style='text-align:center;'>
<img src="assets/img/metapop.png" width="500px"></img>
</div>

---

# Levins metapopulation model

## Parameters

- c 	: colonization rate (scalar)
- e 	: extinction rate (scalar)
- adjMat: adjacency matrix 
- px 	: vector of occupancies
- d 	: vector of destroyed patches

---

# Levins metapopulation model

## Main model


```r
Levins = function(p,adjMat,c,e,d) {
	# Colonization rate
	C = p%*%(c*adjMat)
	# Destroyed habitats have a 0 colonization rate
	C[d==1] = 0
	# Difference between candidate p and the solution 
	px = C/(C + e) - p
	return(px)
}
```

---

# Levins metapopulation model

## Function to extract equilibrium occupancies


```r
library(rootSolve)
get_px = function(adjMat,c,e,d) {
	# Function to optimize
	p_function = function(p) Levins(p,adjMat,c,e,d)
	# Initial conditions
	Start = numeric(n) + 1 - e/c 
	# Extract roots
	multiroot(p_function, start = Start, positive=TRUE)[[1]]
}
```

---

# Levins metapopulation model

## Example


```r
d = numeric(n)
d[1:10] = 1
eq_px = get_px(adjMat,c=0.12,e=0.1,d=d)
RK = rank(eq_px)
col = heat.colors(n = n)
vec.col = numeric(n)
for(i in 1:n) vec.col[i] = col[RK[i]]
vec.col[d==1] = "black"
points(XY[,1],XY[,2],pch=21,bg=vec.col,cex=2)
```

---

# Levins metapopulation model

<div style='text-align:center;'>
<img src="assets/img/metapop_protected.png" width="500px"></img>
</div>
