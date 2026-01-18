# Finite State Machine (FSM) Simulator

## Project Overview
This project is a C-based state machine simulator developed as part of COE428 – Engineering Algorithms and Data Structures at Toronto Metropolitan University. It simulates a system with multiple states and binary transitions, incorporating advanced features like dynamic configuration and reachability-based garbage collection.

The simulator responds to real-time commands to transition between states, modify the internal structure, and perform memory-safe deletion of unreachable nodes.


## Technical Skills Demonstrated
* Systems Programming (C): Implementation of an executable program (simState) that handles standard I/O and manages complex data structures to maintain state logic.
* Graph Theory & Algorithms: Developed reachability analysis to identify "garbage" states—nodes that are no longer accessible from the current state.
* Dynamic Data Management: Implemented the c (change) command, allowing for the real-time reconfiguration of state transitions during execution.
* Input Validation & Exception Handling: Enforced strict state name constraints (A–H) and protected the system by preventing operations on "deleted" states.

## Features
- **State Transitions:** Updates and prints current state based on binary '0' or '1' inputs.
- **Live Reconfiguration:** Allows modifying the machine's transition table on the fly.
- **Garbage Identification:** Detects states that can no longer be reached from the current state.
- **State Deletion:** Marks unreachable states as "deleted" to optimize the machine configuration.

## How to Run
1. Compile the program using `make` or `gcc`:
   ```bash
   gcc simState.c -o simState
   ```
2. Run the executable:
   ```bash
    ./simState
   ```
### simState
```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <string.h>

/* Define the structure for each state in the machine [cite: 50, 51] */
typedef struct {
    char name;
    int next0;    // Index of state if input is '0'
    int next1;    // Index of state if input is '1'
    bool isDeleted;
    bool isReachable;
} State;

State fsm[8]; // Array representing states A through H
int current = 2; // Initial state 'C' [cite: 53]

/**
 * Perform a reachability analysis starting from the current state[cite: 32].
 */
void updateReachability() {
    for (int i = 0; i < 8; i++) fsm[i].isReachable = false;
    
    int queue[8], head = 0, tail = 0;
    queue[tail++] = current;
    fsm[current].isReachable = true;

    while (head < tail) {
        int v = queue[head++];
        int n0 = fsm[v].next0;
        int n1 = fsm[v].next1;

        if (!fsm[n0].isReachable && !fsm[n0].isDeleted) {
            fsm[n0].isReachable = true;
            queue[tail++] = n0;
        }
        if (!fsm[n1].isReachable && !fsm[n1].isDeleted) {
            fsm[n1].isReachable = true;
            queue[tail++] = n1;
        }
    }
}

int main() {
    // Initializing state names and default transitions (A-H) [cite: 52, 54-57]
    char names[] = {'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H'};
    int init0[] = {1, 0, 0, 3, 4, 5, 6, 7}; // Default 0 transitions
    int init1[] = {2, 0, 3, 2, 4, 5, 6, 7}; // Default 1 transitions

    for (int i = 0; i < 8; i++) {
        fsm[i].name = names[i];
        fsm[i].next0 = init0[i];
        fsm[i].next1 = init1[i];
        fsm[i].isDeleted = false;
    }

    // Starting output [cite: 22]
    printf("%c\n", fsm[current].name);

    char cmd[50];
    while (fgets(cmd, sizeof(cmd), stdin)) {
        if (cmd[0] == '0') {
            current = fsm[current].next0;
            printf("%c\n", fsm[current].name);
        } else if (cmd[0] == '1') {
            current = fsm[current].next1;
            printf("%c\n", fsm[current].name);
        } else if (cmd[0] == 'c') {
            // Change command: c <0/1> <StateName> [cite: 25]
            int bit = cmd[2] - '0';
            int target = cmd[4] - 'A';
            if (target >= 0 && target < 8 && !fsm[target].isDeleted) {
                if (bit == 0) fsm[current].next0 = target;
                else fsm[current].next1 = target;
            }
        } else if (cmd[0] == 'p') {
            // Print command: Displays active configuration [cite: 29, 30]
            for (int i = 0; i < 8; i++) {
                if (!fsm[i].isDeleted) {
                    printf("%c %c %c\n", fsm[i].name, fsm[fsm[i].next0].name, fsm[fsm[i].next1].name);
                }
            }
        } else if (cmd[0] == 'g') {
            // Garbage Identify command [cite: 31, 32]
            updateReachability();
            bool garbageExists = false;
            char garbageList[20] = "";
            for (int i = 0; i < 8; i++) {
                if (!fsm[i].isReachable && !fsm[i].isDeleted) {
                    strncat(garbageList, &fsm[i].name, 1);
                    strcat(garbageList, " ");
                    garbageExists = true;
                }
            }
            if (!garbageExists) printf("No garbage\n");
            else printf("Garbage: %s\n", garbageList);
        } else if (cmd[0] == 'd') {
            // Delete command [cite: 36, 38]
            updateReachability();
            bool deletedAny = false;
            printf("Deleted: ");
            for (int i = 0; i < 8; i++) {
                if (!fsm[i].isReachable && !fsm[i].isDeleted) {
                    fsm[i].isDeleted = true;
                    printf("%c ", fsm[i].name);
                    deletedAny = true;
                }
            }
            if (!deletedAny) printf("No states deleted.");
            printf("\n");
        }
    }
    return 0;
}
```
