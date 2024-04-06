# An approach to solve the Vehicle Routing Problem (VRP) using quantum computing

This repository contains a code of a few quantum computing algorithms for solving VRP (and its variants, e.g., CMDVRP (Capacitated Multi-Depot Vehicle Routing Problem)), based on D-Wave's Leap framework for quantum annealing. Older version of this code was a base for the article "New Hybrid Quantum Annealing Algorithms for Solving Vehicle Routing Problem" https://link.springer.com/chapter/10.1007%2F978-3-030-50433-5_42, where you can find more detailed descriptions of these algorithms.

**NOTICE**: **This example requires a legacy version of Ocean.**

## Usage

### Getting started

Firstly, you need to be able to use D-Wave Ocean platform. In order to do that, follow this documentation https://docs.ocean.dwavesys.com/en/stable/overview/install.html. 

You also need to install all other required packages. In order to do that, you have to run the command "pip3 install -r requirements.txt" in a folder with the file "requirements.txt".

Then, you can start running the solvers (e.g., you can run scripts from the "examples" directory, e.g., python3 SolutionPartitioningSolver.py).

If you want to change something in communication with D-Wave, check DWaveSolvers.py. There you can change solvers' configuration and change 'solve_qubo' function. As long as 'solve_qubo' function returns QUBO solution like all D-Wave's solvers, the rest of the code should work properly. 

### Input

There are two input formats. You can find examples in the 'formats' directory.

#### Full input

The full input needs a graph file and a test file. In a graph file, you need to provide edges description: each line should contain two nodes' ids and cost. Cost is related to what you want to optimize, for example: distance or time. In a test file, you need to provide information about depots, destinations and vehicles. In the first line, there should be the number of depots. In the second line, there should be the IDs of depots separated by spaces. In the next line, there should be the number (N) of IDs of nodes that should be visited. The next N lines should contain IDs of nodes that should be visited (potentially with weights of products that should be delivered, in the case of solving a "capacitated" version of the problem). The next line should contain the number of vehicles. In the case of solving a "capacitated" version of the problem, the next line should contain the capacities of vehices separated by a space. 

If you want to solve an instance of MDVRP (without capacities), you only need to provide depots' and destinations' ids and the number of vehicles. If you want to solve CMDVRP, you also need to provide destinations' weights and vehicles' capacities. 

If you have both files, you can use the function 'read_full_test' (from input.py) to obtain VRPProblem object. Note that if you want to solve MDVRP, you need to set the 'capacity' parameter to False.

Using this format can take long time on big graphs with many depots and destinations, so if you want to use the same problem many times, we recommend to use the second format. You can use the 'create_test' function (from input.py) to generate an input in second ('normal') format from this format.

#### Normal input

It needs only a test file. You need to provide information about depots, destinations, vehicles and costs of travelling between depots and destinations. If you want to solve MDVRP (without capacities), you only need to provide depots' and destinations' ids, number of vehicles and costs. If you want to solve CMDVRP, you also need to provide destinations' weights and vehicles' capacities.

Depots and destinations are enumerated with natural numbers. If you have n depots and m destinations, depots will have numbers 0, 1, 2, ..., n - 1 and destinations will have numbers n, n + 1, n + 2, ..., n + m - 1. You need to provide (n + m) x (n + m) matrix with costs. 

If you have a test file, you can use function 'read_test' (from input.py) to obtain VRPProblem object.

### Solvers

Once you have a VRPProblem object, you need to choose a solver. All solvers have the same 'solve' interface. You just need to provide a VRPProblem object, two constants and information if you want to solve problem on CPU or QPU. You can find more detailed description in vrp_solvers.py. You can find examples of using every solver in the 'examples' directory.

#### FullQuboSolver

It solves MDVRP (and not CMDVRP) by formulating problem as QUBO and solving it with D-Wave's solver. It's the weakest solver that works effectively on problems with max 30 destinations and few (1-3) vehicles. 

#### AveragePartitionSolver

It is a FullQuboSolver, but with additional constraint that each vehicle serves approximately the same number of destinations. There is an additional attribute 'limit_radius', which is a maximum absolute difference between the number of destinations that each vehicle can serve and the average number of destinations that each vehicle serves. For small values of limit_radius, solver works effectively on tests with max 50 destinations.

Note that if you have only one vehicle, this solver works exactly like FullQuboSolver.

#### DBScanSolver

This solver uses more classical approaches (DBSCAN algorithm https://en.wikipedia.org/wiki/DBSCAN). It solves a problem by solving small instances of TSP with FullQuboSolver. 'max_len' attribute is the maximum number of destinations in problems that will be solved by FullQuboSolver. It also has the 'anti_noiser' parameter, which tells the solver if it should get rid of singleton clusters in DBSCAN. It is expected that setting 'anti_noiser' False would work better if there are many isolated destinations.

Note that the default 'max_len' value is 10. It is so small that FullQuboSolver can find the best solution. Experiments show that this value works effectively on tests with 250 destinations. But of course, we encourage to experiment with bigger values of this parameter on different tests.

Also, we want to optimize classical parts of this solver to make bigger experiments possible.

This solver is for MDVRP, but it has implemented a prototype solving CMDVRP, which should work when all capacities are the same.

#### SolutionPartitioningSolver

This is a solver for MDVRP and CMDVRP. It uses another solver ('solver' attribute) to solve TSP, and then tries to divide the solution to consecutive parts that will be served by vehicles. There is also an attribute 'random' - bigger value should give better solutions with bigger execution time.

Using this solver with DBScanSolver should give the best effect. On smaller tests (with the number of destinations up to 50), you can use it with FullQuboSolver to reduce the size of QUBO.

Note that using this solver with AveragePartitionSolver is exactly the same as using it with FullQuboSolver.

### Output

'solve' function of solver returns VRPSolution object, which has a solution attribute - a list of lists of vehicles' paths. The first and last numbers on these lists are depots, and continous destinations between them.

Note that independently on the used input format, depots and destinations are enumerated. Numbers from 0 to (number of depots - 1) are depots and next numbers are destinations.

## Code overview

* all code is in 'src' directory, all other directories contains examples and test files
* DwaveSolvers.py contains interface for our solvers to communicate with D-Wave
* input.py contains functions to read problem instantion from input files
* qubo_helper.py contains Qubo class which helps creating QUBO (i.e. merging two QUBOs) 
* vrp_problem.py contains VRPProblem class which contains informations about problem and provides methods to formule problem as QUBO
* vrp_solvers.py contains our solvers
* vrp_solution.py contains VRPSolution class which represent problems solution

## License
Released under the Apache License 2.0. See [LICENSE](https://github.com/xBorox1/D-Wave-VRP/blob/master/src/LICENSE) file.
