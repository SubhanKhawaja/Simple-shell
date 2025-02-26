# Simple-shell
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

#define MAX_COMMAND 1024
#define MAX_ARGS 64

// Function to read input
void read_command(char *command) {
    printf("myshell> ");
    fgets(command, MAX_COMMAND, stdin);
    command[strcspn(command, "\n")] = 0; // Remove newline
}

// Function to parse command into arguments
void parse_command(char *command, char **args) {
    char *token = strtok(command, " ");
    int i = 0;
    while (token != NULL) {
        args[i++] = token;
        token = strtok(NULL, " ");
    }
    args[i] = NULL;
}

// Function to execute commands
void execute_command(char **args) {
    if (args[0] == NULL) return; // Empty command
    
    if (strcmp(args[0], "exit") == 0) {
        exit(0);
    }
    if (strcmp(args[0], "cd") == 0) {
        if (args[1] == NULL) {
            fprintf(stderr, "cd: missing argument\n");
        } else {
            if (chdir(args[1]) != 0) {
                perror("cd");
            }
        }
        return;
    }
    
    pid_t pid = fork();
    if (pid == 0) { // Child process
        if (execvp(args[0], args) == -1) {
            perror("Error executing command");
        }
        exit(EXIT_FAILURE);
    } else if (pid > 0) { // Parent process
        wait(NULL);
    } else {
        perror("fork failed");
    }
}

int main() {
    char command[MAX_COMMAND];
    char *args[MAX_ARGS];
    
    while (1) {
        read_command(command);
        parse_command(command, args);
        execute_command(args);
    }
    
    return 0;
}
