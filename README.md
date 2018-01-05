# README BooleanDOI

# I)	THE METHOD
This repository contains functions using GRASP algorithm to solve the target control problem in Boolean network model, proposed in a paper titled "Target Control in Logical Models Using the Domain of Influence of Nodes" submitted to the special issue ''Logical Modeling of Cellular Processes'' of Frontiers in Physiology. A preprint version is available at BioRxiv: https://www.biorxiv.org/content/early/2018/01/04/243246 .

This algorithm is based on a concept named logic domain of influence and a random heuristic algorithm called greedy randomized adaptive search procedure (GRASP). More details can be found in the paper [1] and [2].

The code is written in Python 2.7. The module requires NetworkX 1.11.

Correspondence can be directed to yanggangthu@gmail.com. Please cite the corresponding paper when the code is used or adapted.

[1] PARDALOS, P. M., T. QIAN, and M. G. RESENDE (1998) “A Greedy Randomized Adaptive Search Procedure for the Feedback Vertex Set Problem,” Journal of Combinatorial Optimization, 2(4), pp. 399–412.

[2] FESTA, P., P. M. PARDALOS, and M. G. C. RESENDE (2001) “Algorithm 815: FORTRAN Subroutines for Computing Approximate Solutions of Feedback Set Problems Using GRASP,” ACM Trans. Math. Softw., 27(4), pp. 456–464

# II) STRUCTURE OF MODULE
Related functions are stored in BooleanDOI_processing.py, BooleanDOI_DOI.py, BooleanDOI_TargetControl.py and qm.py.

qm.py is an external library for Quine--McCluskey algorithm written by George Prekas <prekgeo@yahoo.com>, whose work is based on code from Robert Dick <dickrp@eecs.umich.edu> and Pat Maupin <pmaupin@gmail.com>.

BooleanDOI_processing.py contains functions to do pre-processing for the target control algorithm. It contains functions that can read and write a Boolean rule file in the format of Booleannet. (More details about Booleannet can be found in https://github.com/ialbert/booleannet ). It also contains functions to calculate the expanded network or reduced network from a Boolean network model constructed through reading a Boolean rule file as mentioned above. In the end, it contains a function to read the expanded network generated by Jorge's stable motif algorithm written in Java. (More details can be found in https://github.com/jgtz/StableMotifs ).

BooleanDOI_DOI.py contains functions to calculate the Logic Domain of Influence (LDOI) of a given node state set on the expanded network.

BooleanDOI_TargetControl.py contains functions to calculate solutions to a Target Control problem (on the expanded network).

example1.py contains an example illustrating how to use the code.

example1.txt is the corresponding Boolean rule file used in example1.py.

# III)	INSTRUCTIONS  
# (a) Logic Domain of Influence
To calculate the Logic Domain of Influence (LDOI) of a given node state set, 
just import BooleanDOI_DOI and call truncated_node_of_influence_BFS(G, source) .

To use this module, import FVS and call FVS() just as a regular function.  
The function can take 6 paramters listed below and only the first (the graph) is neccessary.  

Parameters
----------
G         : the given expanded network as an DiGraph

source    : the list of node states that we are interested in to find LDOI

            source node can either be a single node or a list of nodes,
            
            DO NOT INCLUDE A NODE AND ITS NEGATION FOR SOURCE

Default Parameter Values  
-----------------------
composite_node_delimiter='_' : as name suggested, used to parse a composite node, e.g. a composite node 'A_B' means A AND B for the default value
                               
Negation_func= : the function to calculate the complementary node of a given node on the expanded network

Returns
-------
LDOI in the format of a set,

complementary list for the visited part of the search

a list containing any found conflict name to the source during the search process

a list containing all the composte nodes that is not included in the LDOI

# (b) Target Control
To apply the target control algorithm, 
just import BooleanDOI_TargetControl and call update_single_DOI() and GRASP_target_control().

For the GRASP_target_control function, the following parameters are needed
Parameters
----------
G_expanded: the input expanded_graph as a DiGraph object

Target : the target set

max_itr : the number of iterations that are repeated to find the solution

TDOI_BFS : a dictionary to store the LDOI of each node state set

flagged : a dictionary to store whether the LDOI of each node state set contains the negation of any target node

potential : a dictionary to store the composite nodes visited but not included during the search process for LDOI for each node state set

Default Parameter Values  
-----------------------
candidates_score_index : the chosen heuristic greedy function,

                         '0' represents the size of LDOI, 
                         
                         '1' represents the size of composite nodes attached to the LDOI
                         
                         '2' represents the linear combination of the two above with the equal weight
                         
                         '3' represents the size of LDOI with penalization, penalized by multiplied with -1 for those containing                                      negation of any target
                         
                         '4' represents the size of LDOI with penalization, penalized by being subtracted by maximum LDOI for those                                  containing negation of any target
                         
forbidden_nodes : the nodes that are forbiden to be used, both the positive states and the negative states will be forbidden

avail_nodes : the nodes that are available to be used, both the positive states and the negative states will be available

forbidden_node_states : the node states that are forbiden to be used
  
notice different meaning for the default value for forbidden_nodes and avail_nodes: when forbidden_nodes is empty, we do not forbid any node, when avail_nodes is empyty, we allow all nodes

Returns
-------
solutions: a dictionay maps a tuple to a integer value

           the first element of the tuple is the solution as a sorted tuple
  
           the second element of the tuple is a Boolean value to indicate whether the LDOI of the solution is incompatible or not. True  means incompatible.
           
           The integer number is the frequency this solution is obtained during the max_itr iterations.
           
A Boolean value to indicate whether we found any solution

# IV) EXAMPLES
More details can be found in example1.py

>>>import networkx as nx

>>>import BooleanDOI_processing as BDOIp

>>>import BooleanDOI_TargetControl as BDOItc

Here we construct a Boolean network model from a written Boolean rules in the Booleannet format. 
The rules are store in the text file "example1.txt"

>>>f = open("example1.txt",r)

>>>lines = f.readlines()

>>>f.close()

>>>Gread, readnodes = BDOIp.form_network(lines, sorted_nodename = False)

Then one can obtain the expanded network by calling function Get_expanded_network
>>>G_expanded = BDOIp.Get_expanded_network(Gread)  

Then one can solve a target control problem as followed. First one need to initialize the three dictionary called TDOI_BFS, flagged and potential as a global variable. Then one can call the update_single_DOI to calculate the LDOI for each single node. Then one can call the GRASP_target_control algorithm to solve the target control algorithm.
>>>TDOI_BFS, flagged, potential = {}, {}, {}

>>>BDOItc.update_single_DOI(G_expanded, network_name='example1', TDOI_BFS, flagged, potential)

>>>solutions, solution_found = BDOItc.GRASP_target_control(G_expanded, Target = set(['n2n']), max_itr = 500, TDOI_BFS, flagged, potential, custom_score_index = '3')

>>>print solutions

A sample output looks like 
>>>defaultdict(<type 'int'>, {(('~n5n',), True): 195, (('n4n',), False): 301, (('n3n', 'n5n'), False): 4})


It means that ['~n5n'], ['n4n'], ['n3n','n5n'] are the solutions found, they are incompatible, compatible and compatible solution respectively, and they have been found 195, 301 and 4 times respectively during the 500 iterations.

# V)	COPYRIGHT


The MIT License (MIT)

Copyright (c) 2017 Gang Yang and Reka Albert.

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
