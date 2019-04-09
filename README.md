# NuSMV Model Checker
## Introduction
To use NuSMV for model checking.

## The main purpose of a model checker
* To **verify** that a system model **satisfies** a set of desired properties **(property specification)**.
* **Inputs:** 
* 1. **System Model** represented by **FSMs**.
* 2. **Property Specification** represented by **Temporal Logics**: Computational Tree Logic (CTL), Linear Temporal Logic (LTL).
* **Output:**
* 1. Fail: **counterexample** is generated.
* 2. Pass: No sequence of states that lead to an error has found.
* **Data types of ***state variable*** in the language**:
* 1. booleans: e.g., request: boolean;
* 2. scalars: e.g., state: {ready, busy};
* 3. fixed arrays: e.g., arr: array 0..3 of {red, green, blue};
* 4. bounded integers(intervals): e.g., n: 1..8;


## Getting Started
### Prerequisites
* Download [NuSMV](http://nusmv.fbk.eu/NuSMV/download/getting-v2.html)
### Compilation & Run
* Open command prompt at the **bin** folder where **NuSMV.exe** is located.
 ```
NuSMV -int filename.smv
```
### To create **.smv file** in Windows
```
open con filename.smv
```
* Enter the system model and property specification.
* **ctrl-z** then **enter** to exit and save


## References
* http://nusmv.fbk.eu/NuSMV/userman/v21/nusmv_2.html
* http://nusmv.fbk.eu/NuSMV/tutorial/v25/tutorial.pdf
* [8 queens problem](http://www.di.univr.it/documenti/OccorrenzaIns/matdid/matdid387048.smv)
* http://www.inf.ed.ac.uk/teaching/courses/fv/slides/slides03.pdf
* http://twiki.di.uniroma1.it/pub/MFS/WebHome/MF2.pdf
