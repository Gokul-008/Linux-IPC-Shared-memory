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
```
Developed By: GOKUL M

Reg No: 212222230037
```
## Write a C program that illustrates two processes communicating using shared memory.
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>     // For fork(), sleep()
#include <sys/mman.h>   // For shm_open(), mmap(), munmap()
#include <sys/stat.h>   // For mode constants (S_IRUSR, S_IWUSR)
#include <fcntl.h>      // For O_CREAT, O_RDWR
#include <sys/wait.h>   // For wait()

// --- Constants ---
#define SHM_NAME "/my_shared_memory" // Name for the shared memory object
#define SHM_SIZE 1024               // Size of the shared memory segment (1KB)

int main() {
    int shm_fd;         // Shared memory file descriptor
    void* shm_ptr;      // Pointer to the shared memory segment
    pid_t pid;          // Process ID for fork()

    // --- 1. Create/Open Shared Memory Segment ---
    // shm_open: Creates a new shared memory object or opens an existing one.
    // O_CREAT: Create the object if it doesn't exist.
    // O_RDWR: Open for reading and writing.
    // 0666: Permissions (read/write for owner, group, others).
    shm_fd = shm_open(SHM_NAME, O_CREAT | O_RDWR, 0666);
    if (shm_fd == -1) {
        perror("shm_open failed");
        exit(EXIT_FAILURE);
    }
    printf("Shared memory object '%s' created/opened.\n", SHM_NAME);

    // --- 2. Set the Size of the Shared Memory Segment ---
    // ftruncate: Truncates (sets size) of the file descriptor shm_fd to SHM_SIZE bytes.
    if (ftruncate(shm_fd, SHM_SIZE) == -1) {
        perror("ftruncate failed");
        shm_unlink(SHM_NAME); // Clean up on failure
        exit(EXIT_FAILURE);
    }
    printf("Shared memory size set to %d bytes.\n", SHM_SIZE);

    // --- 3. Map Shared Memory into Process Address Space ---
    // mmap: Maps a file or device into memory.
    // Args:
    //   NULL: Let kernel choose address.
    //   SHM_SIZE: Size of mapping.
    //   PROT_READ | PROT_WRITE: Memory can be read from and written to.
    //   MAP_SHARED: Changes are visible to other processes mapping the same region.
    //   shm_fd: File descriptor of the shared memory object.
    //   0: Offset from the beginning of the object.
    shm_ptr = mmap(0, SHM_SIZE, PROT_READ | PROT_WRITE, MAP_SHARED, shm_fd, 0);
    if (shm_ptr == MAP_FAILED) {
        perror("mmap failed");
        shm_unlink(SHM_NAME); // Clean up on failure
        exit(EXIT_FAILURE);
    }
    printf("Shared memory mapped into address space.\n");

    // --- 4. Fork a Child Process ---
    pid = fork();

    if (pid < 0) {
        perror("fork failed");
        // Clean up shared memory before exiting
        munmap(shm_ptr, SHM_SIZE);
        close(shm_fd);
        shm_unlink(SHM_NAME);
        exit(EXIT_FAILURE);
    }

    if (pid == 0) {
        // --- Child Process (Reader) ---
        printf("\n[Child Process (Reader)] PID: %d\n", getpid());
        printf("[Child Process] Waiting for parent to write...\n");
        sleep(2); // Give parent time to write

        // Read data from shared memory
        char buffer[SHM_SIZE];
        strcpy(buffer, (char*)shm_ptr); // Copy string from shared memory
        printf("[Child Process] Read from shared memory: \"%s\"\n", buffer);

        // Child process cleanup (only unmap, unlink is done by parent)
        munmap(shm_ptr, SHM_SIZE);
        close(shm_fd);
        printf("[Child Process] Shared memory unmapped and closed.\n");
        exit(EXIT_SUCCESS);

    } else {
        // --- Parent Process (Writer) ---
        printf("\n[Parent Process (Writer)] PID: %d\n", getpid());

        // Write data to shared memory
        const char* message = "Hello from the parent process via shared memory!";
        printf("[Parent Process] Writing message to shared memory: \"%s\"\n", message);
        strcpy((char*)shm_ptr, message); // Copy string to shared memory

        // Wait for the child process to finish
        printf("[Parent Process] Waiting for child to finish...\n");
        wait(NULL); // Wait for any child process to terminate
        printf("[Parent Process] Child process finished.\n");

        // --- 5. Clean up Shared Memory (Parent's responsibility) ---
        // munmap: Unmap the shared memory segment.
        munmap(shm_ptr, SHM_SIZE);
        close(shm_fd); // Close the file descriptor
        
        // shm_unlink: Removes the shared memory object name from the system.
        // The shared memory region itself is destroyed when all processes have unmapped it
        // and all open file descriptors to it are closed. Unlinking ensures it's removed
        // from the filesystem.
        if (shm_unlink(SHM_NAME) == -1) {
            perror("shm_unlink failed");
            exit(EXIT_FAILURE);
        }
        printf("[Parent Process] Shared memory unmapped, closed, and unlinked.\n");
    }

    return 0;
}
```






## OUTPUT

<img width="1215" height="527" alt="Screenshot from 2025-10-30 10-33-14" src="https://github.com/user-attachments/assets/d31a3c91-e9db-41e1-8bc0-d520917a7513" />

<img width="596" height="286" alt="Screenshot from 2025-10-30 10-33-56" src="https://github.com/user-attachments/assets/9ff9e924-1416-4c45-a71f-a0fa38b7513b" />





# RESULT:
The program is executed successfully.
