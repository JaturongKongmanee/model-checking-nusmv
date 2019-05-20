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

### Example 11
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
	(status = idle | status = A_min) & balance > 0 : A_min;
	status = idle | status =  A_pls : A_pls;
	(status = A_min | status = A_pls) & balance = balance : idle;
esac;
next (accumulated_amount) :=
case
	(status = idle | status = A_min) & balance > 0 : (accumulated_amount + 1) mod 5;
	status = idle | status =  A_pls : (accumulated_amount + 1) mod 5;
	(status = A_min | status = A_pls) & balance = balance : 0;
esac;
next (balance) :=
case
	status = A_min & balance = balance : (balance - accumulated_amount) mod 10;
	status = A_pls & balance = balance : (balance + accumulated_amount) mod 10;
	TRUE: balance;
esac;
LTLSPEC
	F balance > 0;
```

### Example 12
```javascript
MODULE main
VAR
balance: -10..10;
accumulated_amount: 0..5;
status: { A_min, idle };
action: { withdraw, update };
ASSIGN
init (status) := idle;
init (accumulated_amount) := 0;
init (balance) := 3;
next (status) :=
case
	status = idle & action = withdraw : A_min;	
	status = A_min & action = withdraw : A_min;	
	status = A_min & action = update : idle;
	TRUE : status;
esac;
next (accumulated_amount) :=
case
	status = idle & action = withdraw : (accumulated_amount + 1) mod 5;
	status = A_min & action = withdraw : (accumulated_amount + 1) mod 5;	
	status = A_min & action = update : 0;	
	TRUE : accumulated_amount;
esac;
next (balance) :=
case
	status = idle & action = withdraw : balance;
	status = A_min & action = withdraw : balance;
	status = A_min & action = update : (balance - accumulated_amount) mod 10;	
	TRUE : balance;
esac;
LTLSPEC
	F ( G status = idle & balance > 0);
```

### Result of example 12
```javascript
Trace Description: Simulation Trace
Trace Type: Simulation
-> State: 1.1 <-
      balance = 3
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.2 <-
      balance = 3
      accumulated_amount = 1
      status = A_min
      action = update
-> State: 1.3 <-
      balance = 2
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.4 <-
      balance = 2
      accumulated_amount = 1
      status = A_min
      action = update
-> State: 1.5 <-
      balance = 1
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.6 <-
      balance = 1
      accumulated_amount = 1
      status = A_min
      action = withdraw
-> State: 1.7 <-
      balance = 1
      accumulated_amount = 2
      status = A_min
      action = update
-> State: 1.8 <-
      balance = -1
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.9 <-
      balance = -1
      accumulated_amount = 1
      status = A_min
      action = withdraw
-> State: 1.10 <-
      balance = -1
      accumulated_amount = 2
      status = A_min
      action = withdraw
-> State: 1.11 <-
      balance = -1
      accumulated_amount = 3
      status = A_min
      action = withdraw
-> State: 1.12 <-
      balance = -1
      accumulated_amount = 4
      status = A_min
      action = update
-> State: 1.13 <-
      balance = -5
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.14 <-
      balance = -5
      accumulated_amount = 1
      status = A_min
      action = update
-> State: 1.15 <-
      balance = -6
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.16 <-
      balance = -6
      accumulated_amount = 1
      status = A_min
      action = update
-> State: 1.17 <-
      balance = -7
      accumulated_amount = 0
      status = idle
      action = update
```

### Example 13
```javascript
MODULE main
VAR
balance: -10..10;
accumulated_amount: 0..5;
status: { A_min, idle };
action: { withdraw, update };
ASSIGN
init (status) := idle;
init (accumulated_amount) := 0;
init (balance) := 3;
next (status) :=
case
	status = idle & action = withdraw & balance > 0 : A_min;	
	status = A_min & action = withdraw & balance > 0 : A_min;	
	status = A_min & action = update : idle;
	TRUE : status;
esac;
next (accumulated_amount) :=
case
	status = idle & action = withdraw & balance > 0 : (accumulated_amount + 1) mod 5;
	status = A_min & action = withdraw & balance > 0 : (accumulated_amount + 1) mod 5;	
	status = A_min & action = update : 0;	
	TRUE : accumulated_amount;
esac;
next (balance) :=
case
	status = idle & action = withdraw : balance;
	status = A_min & action = withdraw : balance;
	status = A_min & action = update : (balance - accumulated_amount) mod 10;	
	TRUE : balance;
esac;
LTLSPEC
	F ( G status = idle & balance > 0);
```

### Result of example 13
```javascript
Trace Description: Simulation Trace
Trace Type: Simulation
-> State: 1.1 <-
      balance = 3
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.2 <-
      balance = 3
      accumulated_amount = 1
      status = A_min
      action = update
-> State: 1.3 <-
      balance = 2
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.4 <-
      balance = 2
      accumulated_amount = 1
      status = A_min
      action = update
-> State: 1.5 <-
      balance = 1
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.6 <-
      balance = 1
      accumulated_amount = 1
      status = A_min
      action = withdraw
-> State: 1.7 <-
      balance = 1
      accumulated_amount = 2
      status = A_min
      action = update
-> State: 1.8 <-
      balance = -1
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.9 <-
      balance = -1
      accumulated_amount = 0
      status = idle
      action = withdraw
```


### Example 14
```javascript
MODULE main
VAR
balance: -10..10;
accumulated_amount: 0..5;
status: { A_min, idle };
action: { withdraw, update };
ASSIGN
init (status) := idle;
init (accumulated_amount) := 0;
init (balance) := 3;
next (status) :=
case
	status = idle & action = withdraw & balance >= accumulated_amount : A_min;	
	status = A_min & action = withdraw & balance >= accumulated_amount : A_min;	
	status = A_min & action = update : idle;
	TRUE : status;
esac;
next (accumulated_amount) :=
case
	status = idle & action = withdraw & balance >= accumulated_amount : (accumulated_amount + 1) mod 5;
	status = A_min & action = withdraw & balance >= accumulated_amount : (accumulated_amount + 1) mod 5;	
	status = A_min & action = update : 0;	
	TRUE : accumulated_amount;
esac;
next (balance) :=
case
	status = idle & action = withdraw : balance;
	status = A_min & action = withdraw : balance;
	status = A_min & action = update : (balance - accumulated_amount) mod 10;	
	TRUE : balance;
esac;
LTLSPEC
	F ( G status = idle & balance > 0);
```
### Result of example 14
```javascript
Trace Description: Simulation Trace
Trace Type: Simulation
-> State: 1.1 <-
      balance = 3
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.2 <-
      balance = 3
      accumulated_amount = 1
      status = A_min
      action = update
-> State: 1.3 <-
      balance = 2
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.4 <-
      balance = 2
      accumulated_amount = 1
      status = A_min
      action = update
-> State: 1.5 <-
      balance = 1
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.6 <-
      balance = 1
      accumulated_amount = 1
      status = A_min
      action = withdraw
-> State: 1.7 <-
      balance = 1
      accumulated_amount = 2
      status = A_min
      action = update
-> State: 1.8 <-
      balance = -1
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.9 <-
      balance = -1
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.10 <-
      balance = -1
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.11 <-
      balance = -1
      accumulated_amount = 0
      status = idle
      action = withdraw
```

### Example 15
```javascript
MODULE main
VAR
balance: -10..10;
accumulated_amount: 0..5;
status: { A_min, A_pls, idle };
action: { withdraw, update, deposit };
ASSIGN
init (status) := idle;
init (accumulated_amount) := 0;
init (balance) := 3;
next (status) :=
case
	status = idle & action = withdraw & balance >= accumulated_amount : A_min;	
	status = A_min & action = withdraw & balance >= accumulated_amount : A_min;	
	(status = A_min | status = A_pls) & action = update : idle;
	status = idle & action = deposit : A_pls;	
	status = A_pls & action = deposit : A_pls;	
	TRUE : status;
esac;
next (accumulated_amount) :=
case
	status = idle & action = withdraw & balance >= accumulated_amount : (accumulated_amount + 1) mod 5;
	status = A_min & action = withdraw & balance >= accumulated_amount : (accumulated_amount + 1) mod 5;	
	(status = A_min | status = A_pls) & action = update : 0;	
	status = idle & action = deposit : (accumulated_amount + 1) mod 5;	
	status = A_pls & action = deposit : (accumulated_amount + 1) mod 5;
	TRUE : accumulated_amount;
esac;
next (balance) :=
case
	status = idle & action = withdraw : balance;
	status = A_min & action = withdraw : balance;
	status = A_min & action = update : (balance - accumulated_amount) mod 10;
	status = A_pls & action = update : (balance + accumulated_amount) mod 10;
	status = idle & action = deposit : balance;	
	status = A_pls & action = deposit : balance;
	TRUE : balance;
esac;
LTLSPEC
	F ( G status = idle & balance > 0);

```


### Result of example 15
```javascript
Trace Description: Simulation Trace
Trace Type: Simulation
-> State: 1.1 <-
      balance = 3
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.2 <-
      balance = 3
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.3 <-
      balance = 3
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.4 <-
      balance = 3
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.5 <-
      balance = 3
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.6 <-
      balance = 3
      accumulated_amount = 0
      status = idle
      action = deposit
-> State: 1.7 <-
      balance = 3
      accumulated_amount = 1
      status = A_pls
      action = withdraw
-> State: 1.8 <-
      balance = 3
      accumulated_amount = 1
      status = A_pls
      action = update
-> State: 1.9 <-
      balance = 4
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.10 <-
      balance = 4
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.11 <-
      balance = 4
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.12 <-
      balance = 4
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.13 <-
      balance = 4
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.14 <-
      balance = 4
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.15 <-
      balance = 4
      accumulated_amount = 0
      status = idle
      action = deposit
-> State: 1.16 <-
      balance = 4
      accumulated_amount = 1
      status = A_pls
      action = update
-> State: 1.17 <-
      balance = 5
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.18 <-
      balance = 5
      accumulated_amount = 1
      status = A_min
      action = update
-> State: 1.19 <-
      balance = 4
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.20 <-
      balance = 4
      accumulated_amount = 1
      status = A_min
      action = update
-> State: 1.21 <-
      balance = 3
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.22 <-
      balance = 3
      accumulated_amount = 1
      status = A_min
      action = withdraw
-> State: 1.23 <-
      balance = 3
      accumulated_amount = 2
      status = A_min
      action = update
-> State: 1.24 <-
      balance = 1
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.25 <-
      balance = 1
      accumulated_amount = 1
      status = A_min
      action = update
-> State: 1.26 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.27 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.28 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.29 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = deposit
-> State: 1.30 <-
      balance = 0
      accumulated_amount = 1
      status = A_pls
      action = update
-> State: 1.31 <-
      balance = 1
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.32 <-
      balance = 1
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.33 <-
      balance = 1
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.34 <-
      balance = 1
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.35 <-
      balance = 1
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.36 <-
      balance = 1
      accumulated_amount = 1
      status = A_min
      action = update
-> State: 1.37 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.38 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.39 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.40 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = deposit
-> State: 1.41 <-
      balance = 0
      accumulated_amount = 1
      status = A_pls
      action = update
-> State: 1.42 <-
      balance = 1
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.43 <-
      balance = 1
      accumulated_amount = 1
      status = A_min
      action = update
-> State: 1.44 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.45 <-
      balance = 0
      accumulated_amount = 1
      status = A_min
      action = update
-> State: 1.46 <-
      balance = -1
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.47 <-
      balance = -1
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.48 <-
      balance = -1
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.49 <-
      balance = -1
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.50 <-
      balance = -1
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.51 <-
      balance = -1
      accumulated_amount = 0
      status = idle
      action = deposit
-> State: 1.52 <-
      balance = -1
      accumulated_amount = 1
      status = A_pls
      action = update
-> State: 1.53 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.54 <-
      balance = 0
      accumulated_amount = 1
      status = A_min
      action = update
-> State: 1.55 <-
      balance = -1
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.56 <-
      balance = -1
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.57 <-
      balance = -1
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.58 <-
      balance = -1
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.59 <-
      balance = -1
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.60 <-
      balance = -1
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.61 <-
      balance = -1
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.62 <-
      balance = -1
      accumulated_amount = 0
      status = idle
      action = deposit
-> State: 1.63 <-
      balance = -1
      accumulated_amount = 1
      status = A_pls
      action = update
-> State: 1.64 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = update
```

### Example 16
```javascript
MODULE main
VAR
balance: -10..10;
accumulated_amount: 0..5;
status: { A_min, A_pls, idle };
action: { withdraw, update, deposit };
ASSIGN
init (status) := idle;
init (accumulated_amount) := 0;
init (balance) := 3;
next (status) :=
case
	status = idle & action = withdraw & balance > 0 : A_min;	
	status = A_min & action = withdraw & balance > 0 : A_min;	
	(status = A_min | status = A_pls) & action = update : idle;
	status = idle & action = deposit : A_pls;	
	status = A_pls & action = deposit : A_pls;	
	TRUE : status;
esac;
next (accumulated_amount) :=
case
	status = idle & action = withdraw & balance > 0 : (accumulated_amount + 1) mod 5;
	status = A_min & action = withdraw & balance > 0 : (accumulated_amount + 1) mod 5;	
	(status = A_min | status = A_pls) & action = update : 0;	
	status = idle & action = deposit : (accumulated_amount + 1) mod 5;	
	status = A_pls & action = deposit : (accumulated_amount + 1) mod 5;
	TRUE : accumulated_amount;
esac;
next (balance) :=
case
	status = idle & action = withdraw : balance;
	status = A_min & action = withdraw : balance;
	status = A_min & action = update : (balance - accumulated_amount) mod 10;
	status = A_pls & action = update : (balance + accumulated_amount) mod 10;
	status = idle & action = deposit : balance;	
	status = A_pls & action = deposit : balance;
	TRUE : balance;
esac;
LTLSPEC
	F ( G status = idle & balance > 0);
```

### Result of example 16 (Finalized)
```javascript
Trace Description: Simulation Trace
Trace Type: Simulation
-> State: 1.1 <-
      balance = 3
      accumulated_amount = 0
      status = idle
      action = update
.
.
.
-> State: 1.7 <-
      balance = 3
      accumulated_amount = 0
      status = idle
      action = deposit
-> State: 1.8 <-
      balance = 3
      accumulated_amount = 1
      status = A_pls
      action = update
-> State: 1.9 <-
      balance = 4
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.10 <-
      balance = 4
      accumulated_amount = 0
      status = idle
      action = update
.
.
.
-> State: 1.15 <-
      balance = 4
      accumulated_amount = 0
      status = idle
      action = deposit
-> State: 1.16 <-
      balance = 4
      accumulated_amount = 1
      status = A_pls
      action = update
-> State: 1.17 <-
      balance = 5
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.18 <-
      balance = 5
      accumulated_amount = 1
      status = A_min
      action = update
-> State: 1.19 <-
      balance = 4
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.20 <-
      balance = 4
      accumulated_amount = 1
      status = A_min
      action = update
-> State: 1.21 <-
      balance = 3
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.22 <-
      balance = 3
      accumulated_amount = 1
      status = A_min
      action = withdraw
-> State: 1.23 <-
      balance = 3
      accumulated_amount = 2
      status = A_min
      action = update
-> State: 1.24 <-
      balance = 1
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.25 <-
      balance = 1
      accumulated_amount = 1
      status = A_min
      action = update
-> State: 1.26 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.27 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.28 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.29 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = deposit
-> State: 1.30 <-
      balance = 0
      accumulated_amount = 1
      status = A_pls
      action = update
-> State: 1.31 <-
      balance = 1
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.32 <-
      balance = 1
      accumulated_amount = 0
      status = idle
      action = update
.
.
.
-> State: 1.98 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = deposit
-> State: 1.99 <-
      balance = 0
      accumulated_amount = 1
      status = A_pls
      action = update
-> State: 1.100 <-
      balance = 1
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.101 <-
      balance = 1
      accumulated_amount = 1
      status = A_min
      action = withdraw
-> State: 1.102 <-
      balance = 1
      accumulated_amount = 2
      status = A_min
      action = withdraw
-> State: 1.103 <-
      balance = 1
      accumulated_amount = 3
      status = A_min
      action = update
-> State: 1.104 <-
      balance = -2
      accumulated_amount = 0
      status = idle
      action = update
```

### Result of example 16
```javascript
Trace Description: Simulation Trace
Trace Type: Simulation
-> State: 1.1 <-
      balance = 3
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.2 <-
      balance = 3
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.3 <-
      balance = 3
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.4 <-
      balance = 3
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.5 <-
      balance = 3
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.6 <-
      balance = 3
      accumulated_amount = 0
      status = idle
      action = deposit
-> State: 1.7 <-
      balance = 3
      accumulated_amount = 1
      status = A_pls
      action = withdraw
-> State: 1.8 <-
      balance = 3
      accumulated_amount = 1
      status = A_pls
      action = update
-> State: 1.9 <-
      balance = 4
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.10 <-
      balance = 4
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.11 <-
      balance = 4
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.12 <-
      balance = 4
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.13 <-
      balance = 4
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.14 <-
      balance = 4
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.15 <-
      balance = 4
      accumulated_amount = 0
      status = idle
      action = deposit
-> State: 1.16 <-
      balance = 4
      accumulated_amount = 1
      status = A_pls
      action = update
-> State: 1.17 <-
      balance = 5
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.18 <-
      balance = 5
      accumulated_amount = 1
      status = A_min
      action = update
-> State: 1.19 <-
      balance = 4
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.20 <-
      balance = 4
      accumulated_amount = 1
      status = A_min
      action = update
-> State: 1.21 <-
      balance = 3
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.22 <-
      balance = 3
      accumulated_amount = 1
      status = A_min
      action = withdraw
-> State: 1.23 <-
      balance = 3
      accumulated_amount = 2
      status = A_min
      action = update
-> State: 1.24 <-
      balance = 1
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.25 <-
      balance = 1
      accumulated_amount = 1
      status = A_min
      action = update
-> State: 1.26 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.27 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.28 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.29 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = deposit
-> State: 1.30 <-
      balance = 0
      accumulated_amount = 1
      status = A_pls
      action = update
-> State: 1.31 <-
      balance = 1
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.32 <-
      balance = 1
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.33 <-
      balance = 1
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.34 <-
      balance = 1
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.35 <-
      balance = 1
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.36 <-
      balance = 1
      accumulated_amount = 1
      status = A_min
      action = update
-> State: 1.37 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.38 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.39 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.40 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = deposit
-> State: 1.41 <-
      balance = 0
      accumulated_amount = 1
      status = A_pls
      action = update
-> State: 1.42 <-
      balance = 1
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.43 <-
      balance = 1
      accumulated_amount = 1
      status = A_min
      action = update
-> State: 1.44 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.45 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.46 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.47 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.48 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.49 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.50 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.51 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = deposit
-> State: 1.52 <-
      balance = 0
      accumulated_amount = 1
      status = A_pls
      action = update
-> State: 1.53 <-
      balance = 1
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.54 <-
      balance = 1
      accumulated_amount = 1
      status = A_min
      action = update
-> State: 1.55 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.56 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.57 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.58 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.59 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.60 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.61 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.62 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = deposit
-> State: 1.63 <-
      balance = 0
      accumulated_amount = 1
      status = A_pls
      action = update
-> State: 1.64 <-
      balance = 1
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.65 <-
      balance = 1
      accumulated_amount = 0
      status = idle
      action = deposit
-> State: 1.66 <-
      balance = 1
      accumulated_amount = 1
      status = A_pls
      action = update
-> State: 1.67 <-
      balance = 2
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.68 <-
      balance = 2
      accumulated_amount = 0
      status = idle
      action = deposit
-> State: 1.69 <-
      balance = 2
      accumulated_amount = 1
      status = A_pls
      action = withdraw
-> State: 1.70 <-
      balance = 2
      accumulated_amount = 1
      status = A_pls
      action = deposit
-> State: 1.71 <-
      balance = 2
      accumulated_amount = 2
      status = A_pls
      action = update
-> State: 1.72 <-
      balance = 4
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.73 <-
      balance = 4
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.74 <-
      balance = 4
      accumulated_amount = 1
      status = A_min
      action = deposit
-> State: 1.75 <-
      balance = 4
      accumulated_amount = 1
      status = A_min
      action = deposit
-> State: 1.76 <-
      balance = 4
      accumulated_amount = 1
      status = A_min
      action = update
-> State: 1.77 <-
      balance = 3
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.78 <-
      balance = 3
      accumulated_amount = 0
      status = idle
      action = deposit
-> State: 1.79 <-
      balance = 3
      accumulated_amount = 1
      status = A_pls
      action = update
-> State: 1.80 <-
      balance = 4
      accumulated_amount = 0
      status = idle
      action = deposit
-> State: 1.81 <-
      balance = 4
      accumulated_amount = 1
      status = A_pls
      action = update
-> State: 1.82 <-
      balance = 5
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.83 <-
      balance = 5
      accumulated_amount = 0
      status = idle
      action = deposit
-> State: 1.84 <-
      balance = 5
      accumulated_amount = 1
      status = A_pls
      action = withdraw
-> State: 1.85 <-
      balance = 5
      accumulated_amount = 1
      status = A_pls
      action = update
-> State: 1.86 <-
      balance = 6
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.87 <-
      balance = 6
      accumulated_amount = 0
      status = idle
      action = deposit
-> State: 1.88 <-
      balance = 6
      accumulated_amount = 1
      status = A_pls
      action = update
-> State: 1.89 <-
      balance = 7
      accumulated_amount = 0
      status = idle
      action = deposit
-> State: 1.90 <-
      balance = 7
      accumulated_amount = 1
      status = A_pls
      action = withdraw
-> State: 1.91 <-
      balance = 7
      accumulated_amount = 1
      status = A_pls
      action = deposit
-> State: 1.92 <-
      balance = 7
      accumulated_amount = 2
      status = A_pls
      action = deposit
-> State: 1.93 <-
      balance = 7
      accumulated_amount = 3
      status = A_pls
      action = update
-> State: 1.94 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.95 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.96 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.97 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.98 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = deposit
-> State: 1.99 <-
      balance = 0
      accumulated_amount = 1
      status = A_pls
      action = update
-> State: 1.100 <-
      balance = 1
      accumulated_amount = 0
      status = idle
      action = withdraw
-> State: 1.101 <-
      balance = 1
      accumulated_amount = 1
      status = A_min
      action = withdraw
-> State: 1.102 <-
      balance = 1
      accumulated_amount = 2
      status = A_min
      action = withdraw
-> State: 1.103 <-
      balance = 1
      accumulated_amount = 3
      status = A_min
      action = deposit
-> State: 1.104 <-
      balance = 1
      accumulated_amount = 3
      status = A_min
      action = update
-> State: 1.105 <-
      balance = -2
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.106 <-
      balance = -2
      accumulated_amount = 0
      status = idle
      action = deposit
-> State: 1.107 <-
      balance = -2
      accumulated_amount = 1
      status = A_pls
      action = withdraw
-> State: 1.108 <-
      balance = -2
      accumulated_amount = 1
      status = A_pls
      action = deposit
-> State: 1.109 <-
      balance = -2
      accumulated_amount = 2
      status = A_pls
      action = withdraw
-> State: 1.110 <-
      balance = -2
      accumulated_amount = 2
      status = A_pls
      action = update
-> State: 1.111 <-
      balance = 0
      accumulated_amount = 0
      status = idle
      action = deposit
-> State: 1.112 <-
      balance = 0
      accumulated_amount = 1
      status = A_pls
      action = update
-> State: 1.113 <-
      balance = 1
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.114 <-
      balance = 1
      accumulated_amount = 0
      status = idle
      action = deposit
-> State: 1.115 <-
      balance = 1
      accumulated_amount = 1
      status = A_pls
      action = update
-> State: 1.116 <-
      balance = 2
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.117 <-
      balance = 2
      accumulated_amount = 0
      status = idle
      action = update
-> State: 1.118 <-
      balance = 2
      accumulated_amount = 0
      status = idle
      action = deposit
-> State: 1.119 <-
      balance = 2
      accumulated_amount = 1
      status = A_pls
      action = deposit
-> State: 1.120 <-
      balance = 2
      accumulated_amount = 2
      status = A_pls
      action = update
```

## References
* http://nusmv.fbk.eu/NuSMV/userman/v21/nusmv_2.html
* http://nusmv.fbk.eu/NuSMV/tutorial/v25/tutorial.pdf
* [8 queens problem](http://www.di.univr.it/documenti/OccorrenzaIns/matdid/matdid387048.smv)
* http://www.inf.ed.ac.uk/teaching/courses/fv/slides/slides03.pdf
* http://twiki.di.uniroma1.it/pub/MFS/WebHome/MF2.pdf
* http://ce.sharif.edu/courses/92-93/2/ce665-1/resources/root/NuSMV_Resources/A%20Simple%20Tutorial%20on%20NuSMV_Slides.pdf
* http://disi.unitn.it/~agiordani/fm/L5/main.pdf
