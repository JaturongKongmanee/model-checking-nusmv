# NuSMV Model Checker
## Introduction
To use NuSMV for model checking.

## The main purpose of a model checker
* To **verify** that a system model **satisfies** a set of desired properties **(property specification)**.
* **Inputs:** 
	- **System Model** represented by **FSMs**.
	- **Property Specification** represented by **Temporal Logics**: Computational Tree Logic (CTL), Linear Temporal Logic (LTL).
* **Output:**
	- Fail: **counterexample** is generated.
	- Pass: No sequence of states that lead to an error has found.
* **Data types of ***state variable*** in the language**:
	- **booleans**: e.g., request: boolean;
	- **scalars**: e.g., state: {ready, busy};
	- **fixed arrays**: e.g., arr: array 0..3 of {red, green, blue};
	- **bounded integers(intervals)**: e.g., n: 1..8;
	<br/>
* **NOTE!** The **space of states** of the FSM is determined by the **declarations of state variables**.<br/>
[state space is the cartesian product of ranges of state variables.]


## An SMV program consists of:
* Declarations of the **state variables** (state variables determine state space of model).
* Assignments that define the valid initial states.
* Assignments that define the transition relation.


## Getting Started
### Prerequisites
* Download [NuSMV](http://nusmv.fbk.eu/NuSMV/download/getting-v2.html)
### Compilation & Run
* Open command prompt at the **bin** folder where **NuSMV.exe** and **filename.smv** are located.
 ```
NuSMV filename.smv
```
### To create **.smv file** in Windows
```
copy con filename.smv
```
* Enter the system model and property specification.
* **ctrl-z** then **enter** to exit and save


### Example 1
```javascript
MODULE main
VAR
balance_updated: boolean;
balance_amount: { greater_equal, less_than };
ASSIGN
init ( balance_amount ):= greater_equal;
next ( balance_amount ):= 
case
	balance_amount = greater_equal : less_than;
	balance_amount = greater_equal : greater_equal;
	balance_amount = less_than : less_than;
	TRUE : { greater_equal, less_than };
esac;
init ( balance_updated ) := TRUE;
next ( balance_updated ):=
case
	balance_amount = greater_equal : TRUE;
	balance_amount = less_than : TRUE;
	TRUE : {TRUE, FALSE};
esac;

-- It should never be the case that balance_amount = greater_equal and balance_updated = FALSE.
SPEC AG ! (balance_amount = greater_equal & balance_updated = FALSE)
-- It should never be the case that balance_amount = less_than and balance_updated = FALSE.
SPEC AG ! (balance_amount = less_than & balance_updated = FALSE)
-- Each occurrence of condition balance_updated = FALSE is followed by an occurrence of condition balance_updated = TRUE.
SPEC AG ( balance_updated = FALSE -> AF balance_updated = TRUE)
```

### Result of example 1
```javascript
-- specification AG !(balance_amount = greater_equal & balance_updated = FALSE)  is true
-- specification AG !(balance_amount = less_than & balance_updated = FALSE)  is true
-- specification AG (balance_updated = FALSE -> AF balance_updated = TRUE)  is true
```

### Example 2
```javascript
MODULE main
VAR
balance_updated: boolean;
balance_amount: { greater_equal, less_than };
ASSIGN
init ( balance_amount ):= greater_equal;
next ( balance_amount ):= 
case
	balance_amount = greater_equal : less_than;
	balance_amount = greater_equal : greater_equal;
	balance_amount = less_than : less_than;
	TRUE : { greater_equal, less_than };
esac;
init ( balance_updated ):= TRUE;

-- It should never be the case that balance_amount = greater_equal and balance_updated = FALSE.
SPEC AG ! (balance_amount = greater_equal & balance_updated = FALSE)
-- It should never be the case that balance_amount = less_than and balance_updated = FALSE.
SPEC AG ! (balance_amount = less_than & balance_updated = FALSE)
-- Each occurrence of condition balance_updated = FALSE is followed by an occurrence of condition balance_updated = TRUE.
SPEC AG ( balance_updated = FALSE -> AF balance_updated = TRUE)
```

### Result of example 2
```javascript
-- specification AG !(balance_amount = greater_equal & balance_updated = FALSE)  is true
-- specification AG !(balance_amount = less_than & balance_updated = FALSE)  is false
-- as demonstrated by the following execution sequence
Trace Description: CTL Counterexample
Trace Type: Counterexample
-> State: 1.1 <-
  balance_updated = TRUE
  balance_amount = greater_equal
-> State: 1.2 <-
  balance_updated = FALSE
  balance_amount = less_than
-- specification AG (balance_updated = FALSE -> AF balance_updated = TRUE)  is false
-- as demonstrated by the following execution sequence
Trace Description: CTL Counterexample
Trace Type: Counterexample
-> State: 2.1 <-
  balance_updated = TRUE
  balance_amount = greater_equal
-- Loop starts here
-> State: 2.2 <-
  balance_updated = FALSE
  balance_amount = less_than
-> State: 2.3 <-

```

### Example 3
```javascript
MODULE main
VAR
balance_updated: boolean;
balance_amount: { greater_equal, less_than };
ASSIGN
init ( balance_amount ):= greater_equal;
next ( balance_amount ):= 
case
	balance_amount = greater_equal : less_than;
	balance_amount = greater_equal : greater_equal;
	balance_amount = less_than : less_than;
	TRUE : { greater_equal, less_than };
esac;
next ( balance_updated ):=
case
	balance_updated = FALSE : TRUE;
	TRUE : {TRUE, FALSE};
esac;

-- It should never be the case that balance_amount = greater_equal and balance_updated = FALSE.
SPEC AG ! (balance_amount = greater_equal & balance_updated = FALSE)
-- It should never be the case that balance_amount = less_than and balance_updated = FALSE.
SPEC AG ! (balance_amount = less_than & balance_updated = FALSE)
-- Each occurrence of condition balance_updated = FALSE is followed by an occurrence of condition balance_updated = TRUE.
SPEC AG ( balance_updated = FALSE -> AF balance_updated = TRUE)
```

### Result of example 3
```javascript
-- specification AG !(balance_amount = greater_equal & balance_updated = FALSE)  is false
-- as demonstrated by the following execution sequence
Trace Description: CTL Counterexample
Trace Type: Counterexample
-> State: 1.1 <-
  balance_updated = FALSE
  balance_amount = greater_equal
-- specification AG !(balance_amount = less_than & balance_updated = FALSE)  is false
-- as demonstrated by the following execution sequence
Trace Description: CTL Counterexample
Trace Type: Counterexample
-> State: 2.1 <-
  balance_updated = FALSE
  balance_amount = greater_equal
-> State: 2.2 <-
  balance_updated = TRUE
  balance_amount = less_than
-> State: 2.3 <-
  balance_updated = FALSE
-- specification AG (balance_updated = FALSE -> AF balance_updated = TRUE)  is true
```

### Example 4
```javascript
MODULE main
VAR
balance_updated: boolean;
balance_amount: { greater_equal, less_than };
ASSIGN
init ( balance_amount ):= greater_equal;
next ( balance_amount ):= 
case
	balance_amount = greater_equal : less_than;
	balance_amount = greater_equal : greater_equal;
	balance_amount = less_than : less_than;
	TRUE : { greater_equal, less_than };
esac;
init ( balance_updated ):= TRUE;
next ( balance_updated ):=
case
	balance_updated = FALSE : TRUE;
	TRUE : {TRUE, FALSE};
esac;

-- It should never be the case that balance_amount = greater_equal and balance_updated = FALSE.
SPEC AG ! (balance_amount = greater_equal & balance_updated = FALSE)
-- It should never be the case that balance_amount = less_than and balance_updated = FALSE.
SPEC AG ! (balance_amount = less_than & balance_updated = FALSE)
-- Each occurrence of condition balance_updated = FALSE is followed by an occurrence of condition balance_updated = TRUE.
SPEC AG ( balance_updated = FALSE -> AF balance_updated = TRUE)
```

### Result of example 4
```javascript
-- specification AG !(balance_amount = greater_equal & balance_updated = FALSE)  is true
-- specification AG !(balance_amount = less_than & balance_updated = FALSE)  is false
-- as demonstrated by the following execution sequence
Trace Description: CTL Counterexample
Trace Type: Counterexample
-> State: 1.1 <-
  balance_updated = TRUE
  balance_amount = greater_equal
-> State: 1.2 <-
  balance_updated = FALSE
  balance_amount = less_than
-- specification AG (balance_updated = FALSE -> AF balance_updated = TRUE)  is true
```


### Example 5
```javascript
MODULE main
VAR
balance_updated: boolean;
balance_amount: { greater_equal, less_than };
ASSIGN
init ( balance_amount ):= greater_equal;
next ( balance_amount ):= 
case
	balance_amount = greater_equal & balance_updated = TRUE : less_than;
	balance_amount = greater_equal & balance_updated = TRUE : greater_equal;
	balance_amount = less_than & balance_updated = TRUE : less_than;
	TRUE : { greater_equal, less_than };
esac;
init ( balance_updated ):= TRUE;
next ( balance_updated ):=
case
	balance_updated = FALSE & balance_amount = greater_equal : TRUE;
	balance_updated = FALSE & balance_amount = less_than : TRUE;
	TRUE : {TRUE, FALSE};
esac;

-- It should never be the case that balance_amount = greater_equal and balance_updated = FALSE.
SPEC AG ! (balance_amount = greater_equal & balance_updated = FALSE)
-- It should never be the case that balance_amount = less_than and balance_updated = FALSE.
SPEC AG ! (balance_amount = less_than & balance_updated = FALSE)
-- Each occurrence of condition balance_updated = FALSE is followed by an occurrence of condition balance_updated = TRUE.
SPEC AG ( balance_updated = FALSE -> AF balance_updated = TRUE)
```

### Result of example 5
```javascript
-- specification AG !(balance_amount = greater_equal & balance_updated = FALSE)  is true
-- specification AG !(balance_amount = less_than & balance_updated = FALSE)  is false
-- as demonstrated by the following execution sequence
Trace Description: CTL Counterexample
Trace Type: Counterexample
-> State: 1.1 <-
  balance_updated = TRUE
  balance_amount = greater_equal
-> State: 1.2 <-
  balance_updated = FALSE
  balance_amount = less_than
-- specification AG (balance_updated = FALSE -> AF balance_updated = TRUE)  is true
```

### Example 6
```javascript
MODULE main
VAR
balance_updated: boolean;
balance_amount: { greater_equal, less_than };
ASSIGN
init ( balance_amount ):= greater_equal;
next ( balance_amount ):= 
case
	balance_amount = greater_equal & balance_updated = TRUE : less_than;
	balance_amount = greater_equal & balance_updated = TRUE : greater_equal;
	balance_amount = less_than & balance_updated = TRUE : less_than;
	TRUE : { greater_equal, less_than };
esac;
init ( balance_updated ):= TRUE;
next ( balance_updated ):=
case
	balance_updated = TRUE & balance_amount = greater_equal : TRUE;
	balance_updated = TRUE & balance_amount = less_than : TRUE;
	TRUE : {TRUE, FALSE};
esac;

-- It should never be the case that balance_amount = greater_equal and balance_updated = FALSE.
SPEC AG ! (balance_amount = greater_equal & balance_updated = FALSE)
-- It should never be the case that balance_amount = less_than and balance_updated = FALSE.
SPEC AG ! (balance_amount = less_than & balance_updated = FALSE)
-- Each occurrence of condition balance_updated = FALSE is followed by an occurrence of condition balance_updated = TRUE.
SPEC AG ( balance_updated = FALSE -> AF balance_updated = TRUE)
```

### Result of example 6
```javascript
-- specification AG !(balance_amount = greater_equal & balance_updated = FALSE)  is true
-- specification AG !(balance_amount = less_than & balance_updated = FALSE)  is true
-- specification AG (balance_updated = FALSE -> AF balance_updated = TRUE)  is true
```

### Example 7
```javascript
MODULE main
VAR
balance_updated: boolean;
balance_amount: { greater_equal, less_than };
ASSIGN
init ( balance_amount ):= greater_equal;
next ( balance_amount ):= 
case
	balance_amount = greater_equal : less_than;
	balance_amount = greater_equal : greater_equal;
	balance_amount = less_than : less_than;
	TRUE : { greater_equal, less_than };
esac;
init ( balance_updated ):= TRUE;
next ( balance_updated ):=
case
	balance_amount = greater_equal : TRUE;
	balance_amount = less_than : TRUE;
	TRUE : {TRUE, FALSE};
esac;

-- It should never be the case that balance_amount = greater_equal and balance_updated = FALSE.
SPEC AG ! (balance_amount = greater_equal & balance_updated = FALSE)
-- It should never be the case that balance_amount = less_than and balance_updated = FALSE.
SPEC AG ! (balance_amount = less_than & balance_updated = FALSE)
-- Each occurrence of condition balance_updated = FALSE is followed by an occurrence of condition balance_updated = TRUE.
SPEC AG ( balance_updated = FALSE -> AF balance_updated = TRUE)
```
### Result of example 7
```javascript
-- specification AG !(balance_amount = greater_equal & balance_updated = FALSE)  is true
-- specification AG !(balance_amount = less_than & balance_updated = FALSE)  is true
-- specification AG (balance_updated = FALSE -> AF balance_updated = TRUE)  is true
```

### Example 8
```javascript
MODULE main
VAR
balance_updated: boolean;
balance_amount: { greater_equal, less_than };
ASSIGN
init ( balance_amount ):= greater_equal;
next ( balance_amount ):= 
case
	balance_amount = greater_equal : less_than;
	balance_amount = greater_equal : greater_equal;
	balance_amount = less_than : less_than;
	TRUE : { greater_equal, less_than };
esac;
init ( balance_updated ):= TRUE;
next ( balance_updated ):=
case
	balance_updated = FALSE : TRUE;
	TRUE : {TRUE, FALSE};
esac;

-- It should never be the case that balance_amount = greater_equal and balance_updated = FALSE.
SPEC AG ! (balance_amount = greater_equal & balance_updated = FALSE)
-- It should never be the case that balance_amount = less_than and balance_updated = FALSE.
SPEC AG ! (balance_amount = less_than & balance_updated = FALSE)
-- Each occurrence of condition balance_updated = FALSE is followed by an occurrence of condition balance_updated = TRUE.
SPEC AG ( balance_updated = FALSE -> AF balance_updated = TRUE)
```

### Result of example 8
```javascript
-- specification AG !(balance_amount = greater_equal & balance_updated = FALSE)  is true
-- specification AG !(balance_amount = less_than & balance_updated = FALSE)  is false
-- as demonstrated by the following execution sequence
Trace Description: CTL Counterexample
Trace Type: Counterexample
-> State: 1.1 <-
  balance_updated = TRUE
  balance_amount = greater_equal
-> State: 1.2 <-
  balance_updated = FALSE
  balance_amount = less_than
-- specification AG (balance_updated = FALSE -> AF balance_updated = TRUE)  is true
```


### Example 9
```javascript
MODULE main
VAR
balance: -10..10;
accumulated_amount: 0..5;
status: { 0, 1, -1 };
ASSIGN
init (status) := -1;
init (accumulated_amount) := 0;
init (balance) := 0;
next (status) :=
case 
	(status = 0 | status = -1): 0;
	status = 1 | status = -1: 1;
	status = 0 | status = 1: -1;
esac;
next (accumulated_amount) :=
case
	(status = 0 | status = -1): ((accumulated_amount + 1) mod 5) + 1;
	status = 1 | status = -1: ((accumulated_amount + 1) mod 5) + 1;
	status = 0 | status = 1: 0;
esac;
next (balance) :=
case
	status = 0: (balance - accumulated_amount) mod 10;
	status = 1: (balance + accumulated_amount) mod 10;
	TRUE: balance;
esac;
LTLSPEC
	F balance > 0;
```


### Result of example 9
```javascript
-- specification  F balance > 0  is false
-- as demonstrated by the following execution sequence
Trace Description: LTL Counterexample
Trace Type: Counterexample
-> State: 1.1 <-
  balance = 0
  accumulated_amount = 0
  status = -1
-- Loop starts here
-> State: 1.2 <-
  accumulated_amount = 2
  status = 0
-> State: 1.3 <-
  balance = -2
  accumulated_amount = 4
-> State: 1.4 <-
  balance = -6
  accumulated_amount = 1
-> State: 1.5 <-
  balance = -7
  accumulated_amount = 3
-> State: 1.6 <-
  balance = 0
  accumulated_amount = 5
-> State: 1.7 <-
  balance = -5
  accumulated_amount = 2
-> State: 1.8 <-
  balance = -7
  accumulated_amount = 4
-> State: 1.9 <-
  balance = -1
  accumulated_amount = 1
-> State: 1.10 <-
  balance = -2
  accumulated_amount = 3
-> State: 1.11 <-
  balance = -5
  accumulated_amount = 5
-> State: 1.12 <-
  balance = 0
  accumulated_amount = 2
```


### Example 10
```javascript
MODULE main
VAR
balance: -10..10;
accumulated_amount: 0..5;
status: { A_min, A_pls, idle };
ASSIGN
init (status) := idle;
init (accumulated_amount) := 0;
init (balance) := 5;
next (status) :=
case 
	(status = A_min | status = idle) & balance > 0 : A_min;
	status = A_pls | status = idle : A_pls;
	status = A_min | status = A_pls : idle;
esac;
next (accumulated_amount) :=
case
	(status = A_min | status = idle) & balance > 0 : (accumulated_amount + 1) mod 5;
	status = A_pls | status = idle : (accumulated_amount + 1) mod 5;
	status = A_min | status = A_pls : 0;
esac;
next (balance) :=
case
	status = A_min : (balance - accumulated_amount) mod 10;
	status = A_pls : (balance + accumulated_amount) mod 10;
	TRUE: balance;
esac;
LTLSPEC
	F balance > 0;
```


### Result of example 10
```javascript
-- specification  F balance > 0  is true
```

## References
* http://nusmv.fbk.eu/NuSMV/userman/v21/nusmv_2.html
* http://nusmv.fbk.eu/NuSMV/tutorial/v25/tutorial.pdf
* [8 queens problem](http://www.di.univr.it/documenti/OccorrenzaIns/matdid/matdid387048.smv)
* http://www.inf.ed.ac.uk/teaching/courses/fv/slides/slides03.pdf
* http://twiki.di.uniroma1.it/pub/MFS/WebHome/MF2.pdf
* http://ce.sharif.edu/courses/92-93/2/ce665-1/resources/root/NuSMV_Resources/A%20Simple%20Tutorial%20on%20NuSMV_Slides.pdf
* http://disi.unitn.it/~agiordani/fm/L5/main.pdf
