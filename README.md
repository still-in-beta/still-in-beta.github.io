# TITLE: Beefy Parallel DFA Solver, Elli Stoyanova
URL: still-in-beta.github.io
# SUMMARY
I will be solving discrete finite automata in parallel on the NCSA machines CPUs. In particular, I will be implementing and building upon work done in paper “Data-Parallel Finite-State Machines” to explore efficient ways to parallelize the work of DFAs.
# BACKGROUND
Discrete Finite Automata (or DFAs) are finite state machines that either accept or reject a series of symbols in a string by sequentially going through the string and tracking the state. For example, the DFA that accepts the string “aba” over the a-z alphabet would look like:
<img width="418" height="213" alt="image" src="https://github.com/user-attachments/assets/e13410fe-f3d4-4e0f-a7bf-74db87b16a64" />
The string “aba” would take us to state 1, then 2, then accept. The string “ab” would go to state 2, but not accept because there is no a after the b. The string “aacaba” would go to the 1 state, go back to the 0 state after the c, and then go through “aba” and accept.
This work is heavily sequential, as each character is dependent on the previous. However, a simple way to effectively split up work would be to split the string into the number of threads, and track the state at which each string is. For example, on two threads, the string “aaababac” could be split into two, and the aba clause would be found in the second half of the string. This however can get tricky if the length of the regex is larger than the size of the part of the string, since we do not know the previous state. 
One solution, which will be implemented, is the “speculative execution” of the states relative to the current thread. For the string “aaabbaac”, this would look like running the second half of the string from state 0, 1, and 2. Hence, rather than having to wait to get “aaab”’s state, we execute all speculative threads on “baac” and then match aaab’s state to the results. This, of course, depends on communication and whether or not it is worth executing certain speculative threads, which leads us to the challenge.
# THE CHALLENGE
The workload consists of heavy dependencies between different characters. If the pattern is found within a thread calculating a portion of the DFA, the DFA is solved and the program can immediately return “accept”-- this is especially useful on really large strings. However, if the accept string is located between two different threads, there is going to be some extra computation required. As mentioned previously, threads will need to communicate in order to share the state amongst each other. Thread 1 solving the second half of the DFA would need to receive Thread 1’s state. Thread 1 might not be done executing its speculative results before Thread 0 has returned its result. For large alphabets, there are many possible threads in a speculative execution that might exceed the computation of simply doing the DFA sequentially. Certain inputs and certain methods of allocating the threads will have differing speedup, and my goal is to optimize this. 
I chose the NCSA machines because they have the ability to execute large amounts of threads. This is useful, as it gives more options to assign the DFA-solving threads to be for smaller parts of the input versus for calculating speculative results. Nevertheless, the costs and benefits of assigning the threads must be weighed. Assigning threads that cover a larger portion of the input but more speculative threads may or may not be better than assigning threads that cover a smaller portion of the input but less speculative threads.
# RESOURCES
As previously mentioned, the NCSA machines ideally will be used to do this assignment, though the GHC machines are also good for testing (and also in case the NCSA machines are down) albeit with less results than the NCSA machines. “Data-Parallel Finite-State Machines” describes a representation of DFAs consisting of pre-computed 2D arrays mapping all states to each other, where new_state = states[curr_state][input] is the simple computation of the next state. Other than this, however, there is not much prior work on this and I will need to program most of this from scratch. I plan to use OpenMP for thread communication.
# GOALS AND DELIVERABLES
The goals are as follows: 
- The overall DFA data structure and functions and a sequential baseline for the run 
- Generating test cases of large strings and DFAs using large and small alphabets, with varying success and number of locations with succeeding strings
- A parallel program with speculation, with tweakable parameters for scheduling threads to be assigned to different parts or speculations of the regex (e.g, whether two threads should execute different parts starting from state ‘a’ or be assigned to one part starting from state ‘a’ and ‘b’).
- The communication between threads along with it
From there, optimizations will be made from the aforementioned paper. This includes:
- Convergence: If a thread starting from state a and a thread starting from state b on the same string end up in the same state at the same position, then they can merge into one thread. 
- SIMD: Implement a pshufb instruction to exploit SIMD to compute the next states in parallel within a thread. Due to the power of the machines, it would be interesting to tinker with the SIMD width, as only width 16  was tested in the paper.
I hope to be able to implement the parallel program as well as the above optimizations. As a stretch goal, I would hope to create an NVIDIA GPU implementation as well, distributing the sequential chunks amongst the warps. 
# PLATFORM OF CHOICE
NCSA machines, and the C language. This is to make the SIMD optimization easier to work with, as well as being able to implement OpenMP since I am familiar with it. The NCSA machines have high thread counts, which can result in interesting conclusions.
# SCHEDULE
Week 1 (by Nov 15): Set up the basic structure of the code, such as relevant data structures and generate and document test cases. Code up the sequential baseline.
Week 2 (by Nov 22): Code up the parallel version with the speculation with the tweakable thread parameters. Run some initial tests.
Week 3 (by Nov 29): Finish up the parallel version, and get the convergence working.
Week 4 (by Dec 6): Get the 16-wide SIMD optimization working, and try to go wider from there.
The remaining time will be used to catch up on missed deadlines or work on the poster, or do the stretch goal. 
