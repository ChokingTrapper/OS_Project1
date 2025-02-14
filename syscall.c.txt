#include <linux/kernel.h>
#include <linux/syscalls.h>
#include <linux/sched.h>
#include <linux/sched/signal.h>
#include <linux/pid.h>
#include <linux/timekeeping.h>

SYSCALL_DEFINE1(get_youngest_child_time, pid_t, pid)
{
    struct task_struct *parent_task;
    struct task_struct *child_task;
    struct task_struct *youngest_child = NULL;
    unsigned long youngest_start_time = 0;
    unsigned long execution_time = 0;

    parent_task = find_task_by_vpid(pid);
    if (!parent_task) {
        printk(KERN_INFO "Invalid PID: %d\n", pid);
        return -EINVAL; // Return error if PID is not found
    }

    // Iterate over child processes
    list_for_each_entry(child_task, &parent_task->children, sibling) {
        if (!youngest_child || child_task->start_time > youngest_start_time) {
            youngest_child = child_task;
            youngest_start_time = child_task->start_time;
        }
    }

    if (!youngest_child) {
        printk(KERN_INFO "PID %d has no children\n", pid);
        return -ECHILD; // No children found
    }

    struct timespec64 now;
    ktime_get_real_ts64(&now);

    execution_time = now.tv_sec - youngest_child->start_time.tv_sec;

    printk(KERN_INFO "Youngest child of PID %d is PID %d, execution time: %lu seconds\n",
           pid, youngest_child->pid, execution_time);

    return execution_time;
}
