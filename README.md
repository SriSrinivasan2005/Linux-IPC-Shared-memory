# Linux-IPC-Shared-memory
Ex06-Linux IPC-Shared-memory

# AIM:
To Write a C program that illustrates two processes communicating using shared memory.

# DESIGN STEPS:

### Step 1:

Navigate to any Linux environment installed on the system or installed inside a virtual environment like virtual box/vmware or online linux JSLinux (https://bellard.org/jslinux/vm.html?url=alpine-x86.cfg&mem=192) or docker.

### Step 2:

Write the C Program using Linux Process API - Shared Memory

### Step 3:

Execute the C Program for the desired output. 

# PROGRAM:

## Write a C program that illustrates two processes communicating using shared memory.
```
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <sys/shm.h>

#define TEXT_SZ 2048

struct shared_use_st {
    int written_by_you;
    char some_text[TEXT_SZ];
};

int main() {
    int running = 1;
    void *shared_memory = (void *)0;
    struct shared_use_st *shared_stuff;
    char buffer[BUFSIZ];
    int shmid;

    // Create shared memory
    shmid = shmget((key_t)1234, sizeof(struct shared_use_st), 0666 | IPC_CREAT);
    if (shmid == -1) {
        fprintf(stderr, "shmget failed\n");
        exit(EXIT_FAILURE);
    }
    printf("Shared memory ID = %d\n", shmid);

    // Attach shared memory
    shared_memory = shmat(shmid, (void *)0, 0);
    if (shared_memory == (void *)-1) {
        fprintf(stderr, "shmat failed\n");
        exit(EXIT_FAILURE);
    }

    // ✅ Use %p for pointer (portable and safe)
    printf("Memory attached at %p\n", shared_memory);

    shared_stuff = (struct shared_use_st *)shared_memory;

    // Main loop
    while (running) {
        while (shared_stuff->written_by_you == 1) {
            sleep(1);
            printf("Waiting for client...\n");
        }

        printf("Enter some text: ");
        fgets(buffer, BUFSIZ, stdin);

        strncpy(shared_stuff->some_text, buffer, TEXT_SZ);
        shared_stuff->written_by_you = 1;

        if (strncmp(buffer, "end", 3) == 0) {
            running = 0;
        }
    }

    // Detach shared memory
    if (shmdt(shared_memory) == -1) {
        fprintf(stderr, "shmdt failed\n");
        exit(EXIT_FAILURE);
    }

    exit(EXIT_SUCCESS);
}

```
```
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <sys/shm.h>

#define TEXT_SZ 2048

struct shared_use_st {
    int written_by_you;
    char some_text[TEXT_SZ];
};

int main() {
    void *shared_memory = (void *)0;
    struct shared_use_st *shared_stuff;
    int shmid;

    // Access shared memory
    shmid = shmget((key_t)1234, sizeof(struct shared_use_st), 0666 | IPC_CREAT);
    if (shmid == -1) {
        fprintf(stderr, "shmget failed\n");
        exit(EXIT_FAILURE);
    }

    // Attach to shared memory
    shared_memory = shmat(shmid, (void *)0, 0);
    if (shared_memory == (void *)-1) {
        fprintf(stderr, "shmat failed\n");
        exit(EXIT_FAILURE);
    }

    printf("Client attached at %p\n", shared_memory);
    shared_stuff = (struct shared_use_st *)shared_memory;

    int running = 1;
    while (running) {
        if (shared_stuff->written_by_you == 1) {
            printf("Server wrote: %s", shared_stuff->some_text);
            shared_stuff->written_by_you = 0;

            if (strncmp(shared_stuff->some_text, "end", 3) == 0) {
                running = 0;
            }
        }
        sleep(1);
    }

    // Detach
    if (shmdt(shared_memory) == -1) {
        fprintf(stderr, "shmdt failed\n");
        exit(EXIT_FAILURE);
    }

    exit(EXIT_SUCCESS);
}
```
## OUTPUT
![Screenshot 2025-05-14 085147](https://github.com/user-attachments/assets/23d6a76d-9d3a-4b32-8bc5-909ae5e8401e)


![Screenshot 2025-05-14 085132](https://github.com/user-attachments/assets/529a50d4-4aa2-4084-b459-a768359cb95e)


## RESULT:
The program is executed successfully.
