#include <stdio.h>
#include <unistd.h>
#include <sys/syscall.h>
#include <sys/types.h>

#define SYS_get_youngest_child_time 451

int main() {
    pid_t pid = getpid(); // Get current process PID

    int result = syscall(SYS_get_youngest_child_time, pid);
    if (result < 0) {
        perror("syscall failed");
    } else {
        printf("Youngest child execution time: %d seconds\n", result);
    }

    return 0;
}
