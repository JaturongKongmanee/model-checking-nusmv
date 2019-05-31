# NuSMV Model Checker
## Introduction
* **Formal Verification** is the act of proving or disproving the correctness of intended algorithms underlying a system with respect to a certain formal specification or property, using formal methods of mathematics.
* **Model checking** is one approach of formal verification.

### The main purpose of a model checking
* To automatically **verify** that a system model behavior **satisfies** a set of desired properties **(property specification)**.
* **Inputs:** 
	- **System Model** represented by **FSMs**.
	- **Property Specification** represented by:
		- **Temporal Logics:**
			- Computational Tree Logic (**CTL**)
			- Linear Temporal Logic (**LTL**).
		- first-order logic
		- failure-divergence (FDR2)
* **Output:**
	- Fail: **counterexample** is generated.
	- Pass: No sequence of states that lead to an error has found.

## NuSMV
NuSMV is a **symbolic model checking**, but also support **bounded model checking.**
### Using NuSMV model checker consists of:
* Declarations of the **state variables** (state variables determine state space of model).
* Assignments that define the valid initial states.
* Assignments that define the transition relation.

### Data types
* **Data types of ***state variable*** in the language**:
	- **booleans**: e.g., request: boolean;
	- **scalars**: e.g., state: {ready, busy};
	- **fixed arrays**: e.g., arr: array 0..3 of {red, green, blue};
	- **bounded integers(intervals)**: e.g., n: 1..8;
	<br/>
* **NOTE!** The **space of states** of the FSM is determined by the **declarations of state variables**.<br/>
[state space is the [cartesian product](http://web.mnstate.edu/peil/MDEV102/U1/S7/Cartesian4.htm) of ranges of state variables.]

### Specification
* Specification in NuSMV is expressed in **temporal logics.**

### Approach
* NuSMV supports both **binary decision diagrams(BDDs)** and **SAT** approaches.



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

### To run NuSMV interactively
```javascript
NuSMV -int filename.smv
go
pick_state -r
simulate -i -k 5
show_traces -v
```
### To check property(s) - find a counterexample
```javascript
NuSMV -int filename.smv
go
pick_state
check_ltlspec -p "G(balance>=0)"
show_traces -v
```

### A simple bank contract in Solidity
```javascript
pragma solidity >=0.4.22 <0.6.0;
contract Bank {
   mapping (address ⇒ uint8) balances;
   function deposit() {
      balances[msg.sender] += msg.value;
   }
   function withdraw(uint8 _amount) {
      if(balances[msg.sender] ≥ _amount) {
         msg.sender.call.value(_amount)();
	 balances[msg.sender] −= _amount;
      }
   }
   function balance() constant returns(uint8) {
      return balances[msg.sender];
   }
}
```


## Example and Result


<details>
  <summary><b>Final Model used in the paper</b></summary>
  
 ```javascript
MODULE main
VAR
balance: -10..10;
accumulated_amount: 0..5;
status: { A_min, A_pls, idle };

IVAR
action: { withdraw, update, deposit };

ASSIGN
init (status) := idle;
init (accumulated_amount) := 0;
init (balance) := 0;
next (status) :=
case
	(status = idle | status = A_min) & action = withdraw & balance > 0 : A_min;		
	(status = A_min | status = A_pls) & action = update : idle;
	(status = idle | status = A_pls) & action = deposit : A_pls;		
	TRUE : status;
esac;
next (accumulated_amount) :=
case
	(status = idle | status = A_min) & action = withdraw & balance > 0 : (accumulated_amount + 1) mod 5;	
	(status = A_min | status = A_pls) & action = update : 0;	
	(status = idle | status = A_pls) & action = deposit : (accumulated_amount + 1) mod 5;	
	TRUE : accumulated_amount;
esac;
next (balance) :=
case
	(status = idle | status = A_min) & action = withdraw & balance > 0 : balance;
	status = A_pls & action = update : (balance + accumulated_amount) mod 10;
	status = A_min & action = update : (balance - accumulated_amount) mod 10;
	(status = idle | status = A_pls) & action = deposit : balance;	
	TRUE : balance;
esac;
LTLSPEC
	G balance >= 0;
```
</details>

<!--##############################################################################################-->

<details>
  <summary><b>Counter Example 1</b></summary>
  
 ```javascript
NuSMV > check_ltlspec -p "G(balance>=0)"
-- specification  G balance >= 0  is false
-- as demonstrated by the following execution sequence
Trace Description: LTL Counterexample
Trace Type: Counterexample
-- Loop starts here
-> State: 2.1 <-
  balance = 0
  accumulated_amount = 0
  status = idle
-> Input: 2.2 <-
  action = deposit
-> State: 2.2 <-
  accumulated_amount = 1
  status = A_pls
-> Input: 2.3 <-
  action = update
-> State: 2.3 <-
  balance = 1
  accumulated_amount = 0
  status = idle
-> Input: 2.4 <-
  action = withdraw
-> State: 2.4 <-
  accumulated_amount = 1
  status = A_min
-> Input: 2.5 <-
-> State: 2.5 <-
  accumulated_amount = 2
-> Input: 2.6 <-
  action = update
-> State: 2.6 <-
  balance = -1
  accumulated_amount = 0
  status = idle
-> Input: 2.7 <-
  action = deposit
-> State: 2.7 <-
  accumulated_amount = 1
  status = A_pls
-> Input: 2.8 <-
  action = update
-> State: 2.8 <-
  balance = 0
  accumulated_amount = 0
  status = idle
  
  
  
  
<!-- ################### Trace number: 2 ################### -->
Trace Description: LTL Counterexample
Trace Type: Counterexample
-- Loop starts here
-> State: 2.1 <-
      balance = 0
      accumulated_amount = 0
      status = idle
-> Input: 2.2 <-
      action = deposit
-> State: 2.2 <-
      balance = 0
      accumulated_amount = 1
      status = A_pls
-> Input: 2.3 <-
      action = update
-> State: 2.3 <-
      balance = 1
      accumulated_amount = 0
      status = idle
-> Input: 2.4 <-
      action = withdraw
-> State: 2.4 <-
      balance = 1
      accumulated_amount = 1
      status = A_min
-> Input: 2.5 <-
      action = withdraw
-> State: 2.5 <-
      balance = 1
      accumulated_amount = 2
      status = A_min
-> Input: 2.6 <-
      action = update
-> State: 2.6 <-
      balance = -1
      accumulated_amount = 0
      status = idle
-> Input: 2.7 <-
      action = deposit
-> State: 2.7 <-
      balance = -1
      accumulated_amount = 1
      status = A_pls
-> Input: 2.8 <-
      action = update
-> State: 2.8 <-
      balance = 0
      accumulated_amount = 0
      status = idle
```
</details>

<!--##############################################################################################-->

<details>
  <summary><b>Counter Example 2</b></summary>
  
 ```javascript
NuSMV > check_ltlspec -p "G(balance>=0)"
-- specification  G balance >= 0  is false
-- as demonstrated by the following execution sequence
Trace Description: LTL Counterexample
Trace Type: Counterexample
-- Loop starts here
-> State: 2.1 <-
  balance = 2
  accumulated_amount = 0
  status = idle
-> Input: 2.2 <-
  action = withdraw
-> State: 2.2 <-
  accumulated_amount = 1
  status = A_min
-> Input: 2.3 <-
-> State: 2.3 <-
  accumulated_amount = 2
-> Input: 2.4 <-
-> State: 2.4 <-
  accumulated_amount = 3
-> Input: 2.5 <-
  action = update
-> State: 2.5 <-
  balance = -1
  accumulated_amount = 0
  status = idle
-> Input: 2.6 <-
  action = deposit
-> State: 2.6 <-
  accumulated_amount = 1
  status = A_pls
-> Input: 2.7 <-
-> State: 2.7 <-
  accumulated_amount = 2
-> Input: 2.8 <-
-> State: 2.8 <-
  accumulated_amount = 3
-> Input: 2.9 <-
  action = update
-> State: 2.9 <-
  balance = 2
  accumulated_amount = 0
  status = idle
  
  
  
  
  
<!-- ################### Trace number: 2 ################### -->
Trace Description: LTL Counterexample
Trace Type: Counterexample
-- Loop starts here
-> State: 2.1 <-
      balance = 2
      accumulated_amount = 0
      status = idle
-> Input: 2.2 <-
      action = withdraw
-> State: 2.2 <-
      balance = 2
      accumulated_amount = 1
      status = A_min
-> Input: 2.3 <-
      action = withdraw
-> State: 2.3 <-
      balance = 2
      accumulated_amount = 2
      status = A_min
-> Input: 2.4 <-
      action = withdraw
-> State: 2.4 <-
      balance = 2
      accumulated_amount = 3
      status = A_min
-> Input: 2.5 <-
      action = update
-> State: 2.5 <-
      balance = -1
      accumulated_amount = 0
      status = idle
-> Input: 2.6 <-
      action = deposit
-> State: 2.6 <-
      balance = -1
      accumulated_amount = 1
      status = A_pls
-> Input: 2.7 <-
      action = deposit
-> State: 2.7 <-
      balance = -1
      accumulated_amount = 2
      status = A_pls
-> Input: 2.8 <-
      action = deposit
-> State: 2.8 <-
      balance = -1
      accumulated_amount = 3
      status = A_pls
-> Input: 2.9 <-
      action = update
-> State: 2.9 <-
      balance = 2
      accumulated_amount = 0
      status = idle
```
</details>

<!--##############################################################################################-->



<details>
  <summary><b>Counter Example 3</b></summary>
  
 ```javascript
NuSMV > check_ltlspec -p "G(balance>=0)"
-- specification  G balance >= 0  is false
-- as demonstrated by the following execution sequence
Trace Description: LTL Counterexample
Trace Type: Counterexample
-- Loop starts here
-> State: 2.1 <-
  balance = 5
  accumulated_amount = 0
  status = idle
-> Input: 2.2 <-
  action = withdraw
-> State: 2.2 <-
  accumulated_amount = 1
  status = A_min
-> Input: 2.3 <-
-> State: 2.3 <-
  accumulated_amount = 2
-> Input: 2.4 <-
-> State: 2.4 <-
  accumulated_amount = 3
-> Input: 2.5 <-
-> State: 2.5 <-
  accumulated_amount = 4
-> Input: 2.6 <-
  action = update
-> State: 2.6 <-
  balance = 1
  accumulated_amount = 0
  status = idle
-> Input: 2.7 <-
  action = withdraw
-> State: 2.7 <-
  accumulated_amount = 1
  status = A_min
-> Input: 2.8 <-
-> State: 2.8 <-
  accumulated_amount = 2
-> Input: 2.9 <-
  action = update
-> State: 2.9 <-
  balance = -1
  accumulated_amount = 0
  status = idle
-> Input: 2.10 <-
  action = deposit
-> State: 2.10 <-
  accumulated_amount = 1
  status = A_pls
-> Input: 2.11 <-
-> State: 2.11 <-
  accumulated_amount = 2
-> Input: 2.12 <-
  action = update
-> State: 2.12 <-
  balance = 1
  accumulated_amount = 0
  status = idle
-> Input: 2.13 <-
  action = deposit
-> State: 2.13 <-
  accumulated_amount = 1
  status = A_pls
-> Input: 2.14 <-
-> State: 2.14 <-
  accumulated_amount = 2
-> Input: 2.15 <-
-> State: 2.15 <-
  accumulated_amount = 3
-> Input: 2.16 <-
-> State: 2.16 <-
  accumulated_amount = 4
-> Input: 2.17 <-
  action = update
-> State: 2.17 <-
  balance = 5
  accumulated_amount = 0
  status = idle
  
  
  
 
<!-- ################### Trace number: 2 ################### -->
Trace Description: LTL Counterexample
Trace Type: Counterexample
-- Loop starts here
-> State: 2.1 <-
      balance = 5
      accumulated_amount = 0
      status = idle
-> Input: 2.2 <-
      action = withdraw
-> State: 2.2 <-
      balance = 5
      accumulated_amount = 1
      status = A_min
-> Input: 2.3 <-
      action = withdraw
-> State: 2.3 <-
      balance = 5
      accumulated_amount = 2
      status = A_min
-> Input: 2.4 <-
      action = withdraw
-> State: 2.4 <-
      balance = 5
      accumulated_amount = 3
      status = A_min
-> Input: 2.5 <-
      action = withdraw
-> State: 2.5 <-
      balance = 5
      accumulated_amount = 4
      status = A_min
-> Input: 2.6 <-
      action = update
-> State: 2.6 <-
      balance = 1
      accumulated_amount = 0
      status = idle
-> Input: 2.7 <-
      action = withdraw
-> State: 2.7 <-
      balance = 1
      accumulated_amount = 1
      status = A_min
-> Input: 2.8 <-
      action = withdraw
-> State: 2.8 <-
      balance = 1
      accumulated_amount = 2
      status = A_min
-> Input: 2.9 <-
      action = update
-> State: 2.9 <-
      balance = -1
      accumulated_amount = 0
      status = idle
-> Input: 2.10 <-
      action = deposit
-> State: 2.10 <-
      balance = -1
      accumulated_amount = 1
      status = A_pls
-> Input: 2.11 <-
      action = deposit
-> State: 2.11 <-
      balance = -1
      accumulated_amount = 2
      status = A_pls
-> Input: 2.12 <-
      action = update
-> State: 2.12 <-
      balance = 1
      accumulated_amount = 0
      status = idle
-> Input: 2.13 <-
      action = deposit
-> State: 2.13 <-
      balance = 1
      accumulated_amount = 1
      status = A_pls
-> Input: 2.14 <-
      action = deposit
-> State: 2.14 <-
      balance = 1
      accumulated_amount = 2
      status = A_pls
-> Input: 2.15 <-
      action = deposit
-> State: 2.15 <-
      balance = 1
      accumulated_amount = 3
      status = A_pls
-> Input: 2.16 <-
      action = deposit
-> State: 2.16 <-
      balance = 1
      accumulated_amount = 4
      status = A_pls
-> Input: 2.17 <-
      action = update
-> State: 2.17 <-
      balance = 5
      accumulated_amount = 0
      status = idle
```
</details>

<!--##############################################################################################-->




<details>
  <summary><b>Unsafe Model</b></summary>
  
 ```javascript
MODULE main
VAR
balance: -10..10;
accumulated_amount: 0..5;
status: { A_min, A_pls, idle };

IVAR
action: { withdraw, update, deposit };

ASSIGN
init (status) := idle;
init (accumulated_amount) := 0;
init (balance) := 0;
next (status) :=
case
	(status = idle | status = A_min) & action = withdraw & balance > 0 : A_min;		
	(status = A_min | status = A_pls) & action = update : idle;
	(status = idle | status = A_pls) & action = deposit : A_pls;		
	TRUE : status;
esac;
next (accumulated_amount) :=
case
	(status = idle | status = A_min) & action = withdraw & balance > 0 : (accumulated_amount + 1) mod 5;	
	(status = A_min | status = A_pls) & action = update : 0;	
	(status = idle | status = A_pls) & action = deposit : (accumulated_amount + 1) mod 5;	
	TRUE : accumulated_amount;
esac;
next (balance) :=
case
	(status = idle | status = A_min) & action = withdraw & balance > 0 : balance;
	status = A_pls & action = update : (balance + accumulated_amount) mod 10;
	status = A_min & action = update : (balance - accumulated_amount) mod 10;
	(status = idle | status = A_pls) & action = deposit : balance;	
	TRUE : balance;
esac;
LTLSPEC
	G balance >= 0;
```
</details>

<!--##############################################################################################-->


<details>
  <summary><b>Model 1: Scenario 1</b></summary>
  
 ```javascript
- deposit
- deposit
- deposit
- update
- withdraw
- withdraw
- update


Trace Description: Simulation Trace
Trace Type: Simulation
-> State: 1.1 <-
      balance = 0
      accumulated_amount = 0
      status = idle
-> Input: 1.2 <-
      action = deposit
-> State: 1.2 <-
      balance = 0
      accumulated_amount = 1
      status = A_pls
-> Input: 1.3 <-
      action = deposit
-> State: 1.3 <-
      balance = 0
      accumulated_amount = 2
      status = A_pls
-> Input: 1.4 <-
      action = deposit
-> State: 1.4 <-
      balance = 0
      accumulated_amount = 3
      status = A_pls
-> Input: 1.5 <-
      action = update
-> State: 1.5 <-
      balance = 3
      accumulated_amount = 0
      status = idle
-> Input: 1.6 <-
      action = withdraw
-> State: 1.6 <-
      balance = 3
      accumulated_amount = 1
      status = A_min
-> Input: 1.7 <-
      action = withdraw
-> State: 1.7 <-
      balance = 3
      accumulated_amount = 2
      status = A_min
-> Input: 1.8 <-
      action = update
-> State: 1.8 <-
      balance = 1
      accumulated_amount = 0
      status = idle
```
</details>

<!--##############################################################################################-->

<details>
  <summary><b>Model 1: Scenario 2</b></summary>
  
 ```javascript
 - deposit
 - update
 - withdraw
 - withdraw
 - withdraw
 - withdraw
 - update
 - withdraw
 
 
Trace Description: Simulation Trace
Trace Type: Simulation
-> State: 1.1 <-
      balance = 0
      accumulated_amount = 0
      status = idle
-> Input: 1.2 <-
      action = deposit
-> State: 1.2 <-
      balance = 0
      accumulated_amount = 1
      status = A_pls
-> Input: 1.3 <-
      action = update
-> State: 1.3 <-
      balance = 1
      accumulated_amount = 0
      status = idle
-> Input: 1.4 <-
      action = withdraw
-> State: 1.4 <-
      balance = 1
      accumulated_amount = 1
      status = A_min
-> Input: 1.5 <-
      action = withdraw
-> State: 1.5 <-
      balance = 1
      accumulated_amount = 2
      status = A_min
-> Input: 1.6 <-
      action = withdraw
-> State: 1.6 <-
      balance = 1
      accumulated_amount = 3
      status = A_min
-> Input: 1.7 <-
      action = withdraw
-> State: 1.7 <-
      balance = 1
      accumulated_amount = 4
      status = A_min
-> Input: 1.8 <-
      action = update
-> State: 1.8 <-
      balance = -3
      accumulated_amount = 0
      status = idle
-> Input: 1.9 <-
      action = withdraw
-> State: 1.9 <-
      balance = -3
      accumulated_amount = 0
      status = idle
-> Input: 1.10 <-
      action = withdraw
-> State: 1.10 <-
      balance = -3
      accumulated_amount = 0
      status = idle
```
</details>

<!--##############################################################################################-->

<details>
  <summary><b>Model 1: Scenario 3</b></summary>
  
 ```javascript
- deposit
- update
- withdraw
- update
- withdraw
- update
- withdraw
- update

Trace Description: Simulation Trace
Trace Type: Simulation
-> State: 1.1 <-
      balance = 0
      accumulated_amount = 0
      status = idle
-> Input: 1.2 <-
      action = deposit
-> State: 1.2 <-
      balance = 0
      accumulated_amount = 1
      status = A_pls
-> Input: 1.3 <-
      action = update
-> State: 1.3 <-
      balance = 1
      accumulated_amount = 0
      status = idle
-> Input: 1.4 <-
      action = withdraw
-> State: 1.4 <-
      balance = 1
      accumulated_amount = 1
      status = A_min
-> Input: 1.5 <-
      action = update
-> State: 1.5 <-
      balance = 0
      accumulated_amount = 0
      status = idle
-> Input: 1.6 <-
      action = withdraw
-> State: 1.6 <-
      balance = 0
      accumulated_amount = 0
      status = idle
-> Input: 1.7 <-
      action = update
-> State: 1.7 <-
      balance = 0
      accumulated_amount = 0
      status = idle
-> Input: 1.8 <-
      action = withdraw
-> State: 1.8 <-
      balance = 0
      accumulated_amount = 0
      status = idle
-> Input: 1.9 <-
      action = update
-> State: 1.9 <-
      balance = 0
      accumulated_amount = 0
      status = idle
```
</details>

<!--##############################################################################################-->






<details>
  <summary><b>Safe Model : no counterexamle</b></summary>
  
 ```javascript
MODULE main
VAR
balance: -10..10;
accumulated_amount: 0..5;
status: { A_min, A_pls, idle };

IVAR
action: { withdraw, update, deposit };

ASSIGN
init (status) := idle;
init (accumulated_amount) := 0;
init (balance) := 0;
next (status) :=
case
	status = idle & action = withdraw & balance > 0 : A_min;
	status = idle & action = deposit : A_pls;		
	(status = A_min | status = A_pls) & action = update : idle;	
	TRUE : status;
esac;
next (accumulated_amount) :=
case
	status = idle & action = withdraw & balance > 0 : (accumulated_amount + 1) mod 5;
	status = idle & action = deposit : (accumulated_amount + 1) mod 5;		
	(status = A_min | status = A_pls) & action = update : 0;
	TRUE : accumulated_amount;
esac;
next (balance) :=
case
	status = idle & action = withdraw & balance > 0 : balance;
	status = idle & action = deposit : balance;		
	status = A_pls & action = update : (balance + accumulated_amount) mod 10;
	status = A_min & action = update : (balance - accumulated_amount) mod 10;
	TRUE : balance;
esac;
LTLSPEC
	G balance >= 0;
```
</details>


<!--##############################################################################################-->



<details>
  <summary><b>Example 1</b></summary>
  
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
</details>

<!--##############################################################################################-->

<details>
  <summary><b>Result of example 1</b></summary>
  
```javascript
-- specification AG !(balance_amount = greater_equal & balance_updated = FALSE)  is true
-- specification AG !(balance_amount = less_than & balance_updated = FALSE)  is true
-- specification AG (balance_updated = FALSE -> AF balance_updated = TRUE)  is true
```
</details>

<!--##############################################################################################-->

<details>
  <summary><b>Example 2</b></summary>
  
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
</details>

<!--##############################################################################################-->

<details>
  <summary><b>Result of example 2</b></summary>
  
```javascript
-- specification AG !(balance_amount = greater_equal & balance_updated = FALSE)  is true
-- specification AG !(balance_amount = less_than & balance_updated = FALSE)  is true
-- specification AG (balance_updated = FALSE -> AF balance_updated = TRUE)  is true
```
</details>

<!--##############################################################################################-->

<details>
  <summary><b>Example 3</b></summary>
  
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
</details>

<!--##############################################################################################-->

<details>
  <summary><b>Result of example 3</b></summary>
  
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
</details>

<!--##############################################################################################-->


<details>
  <summary><b>Example 4</b></summary>
  
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
</details>

<!--##############################################################################################-->

<details>
  <summary><b>Result of example 4</b></summary>
  
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
</details>

<!--##############################################################################################-->


<details>
  <summary><b>Example 5</b></summary>
  
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
</details>

<!--##############################################################################################-->

<details>
  <summary><b>Result of example 5</b></summary>
  
```javascript
-- specification  F balance > 0  is true
```
</details>

<!--##############################################################################################-->


<details>
  <summary><b>Example 6</b></summary>
  
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
</details>

<!--##############################################################################################-->

<details>
  <summary><b>Result of example 6</b></summary>
  
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
</details>

<!--##############################################################################################-->


<details>
  <summary><b>Example 7</b></summary>
  
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
</details>

<!--##############################################################################################-->

<details>
  <summary><b>Result of example 7</b></summary>
  
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
</details>

<!--##############################################################################################-->


<details>
  <summary><b>Example 8</b></summary>
  
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
</details>

<!--##############################################################################################-->

<details>
  <summary><b>Result of example 8</b></summary>
  
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
</details>

<!--##############################################################################################-->



<details>
  <summary><b>Example 9</b></summary>
  
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
</details>

<!--##############################################################################################-->

<details>
  <summary><b>Result of example 9</b></summary>
  
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
</details>

<!--##############################################################################################-->


<details>
  <summary><b>Example 10</b></summary>
  
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
</details>

<!--##############################################################################################-->

<details>
  <summary><b>Result of example 10 (finalized)</b></summary>
  
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
</details>

<!--##############################################################################################-->

<details>
  <summary><b>Result of example 10</b></summary>
  
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
</details>

<!--##############################################################################################-->


## References
* http://nusmv.fbk.eu/NuSMV/userman/v21/nusmv_2.html
* http://nusmv.fbk.eu/NuSMV/tutorial/v25/tutorial.pdf
* [8 queens problem](http://www.di.univr.it/documenti/OccorrenzaIns/matdid/matdid387048.smv)
* http://www.inf.ed.ac.uk/teaching/courses/fv/slides/slides03.pdf
* http://twiki.di.uniroma1.it/pub/MFS/WebHome/MF2.pdf
* http://ce.sharif.edu/courses/92-93/2/ce665-1/resources/root/NuSMV_Resources/A%20Simple%20Tutorial%20on%20NuSMV_Slides.pdf
* http://disi.unitn.it/~agiordani/fm/L5/main.pdf
* [NuSMV: a new symbolic model checker](http://www.cs.cmu.edu/~emc/papers/Papers%20In%20Refereed%20Journals/NuSMV-ANewSymbolicModelChecker.pdf)
* https://www.cl.cam.ac.uk/teaching/1617/HLog+ModC/slides/lecture-9-4up.pdf
* http://www.tcs.hut.fi/Studies/T-79.5001/reports/laht08simcex.pdf
