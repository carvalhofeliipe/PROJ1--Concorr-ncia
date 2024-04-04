# PROJ1--Concorr-ncia

#define _GNU_SOURCE
#include <stdlib.h>
#include <malloc.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <signal.h>
#include <sched.h>
#include <stdio.h>
#include <pthread.h>

// 64kB stack
#define FIBER_STACK 1024*64
#define MAX_TRANSFERS 100

struct c {
    int saldo;
    pthread_mutex_t lock;
};

typedef struct c conta;

conta from = {100, PTHREAD_MUTEX_INITIALIZER};
conta to = {100, PTHREAD_MUTEX_INITIALIZER};
int valor;

// Função que a thread filha executará
int transferencia(void *arg) {
    pthread_mutex_lock(&from.lock); // Trava o mutex da conta 'from'
    if (from.saldo >= valor) {
        from.saldo -= valor;
        pthread_mutex_unlock(&from.lock); // Destrava o mutex da conta 'from'

        pthread_mutex_lock(&to.lock); // Trava o mutex da conta 'to'
        to.saldo += valor;
        pthread_mutex_unlock(&to.lock); // Destrava o mutex da conta 'to'

        printf("Transferência de %d concluída com sucesso!\n", valor);
        printf("Saldo de c1: %d\n", from.saldo);
        printf("Saldo de c2: %d\n", to.saldo);
    } else {
        pthread_mutex_unlock(&from.lock); // Destrava o mutex da conta 'from' se não houver saldo suficiente
        printf("Não foi possível realizar a transferência de %d. Saldo insuficiente.\n", valor);
        return -1; // Retorna um valor diferente de zero para indicar que a conta 'from' zerou
    }
    return 0;
}

int main() {
    void* stack;
    pid_t pid;
    int i;

    // Aloca a pilha
    stack = malloc(FIBER_STACK);
    if (stack == 0) {
        perror("malloc: could not allocate stack");
        exit(1);
    }

    // Define o saldo inicial para ambas as contas
    from.saldo = 100;
    to.saldo = 100;
    valor = 10;

    printf("Iniciando transferências de 10 para a conta c2\n");

    for (i = 0; i < MAX_TRANSFERS; i++) {
        // Chama a syscall clone para criar a thread filha
        pid = clone(&transferencia, (char*) stack + FIBER_STACK,
        SIGCHLD | CLONE_FS | CLONE_FILES | CLONE_SIGHAND | CLONE_VM, 0);
        if (pid == -1) {
            perror("clone");
            exit(2);
        }
        // Aguarda a thread filha terminar
        int status;
        waitpid(pid, &status, 0);
        if (WEXITSTATUS(status) != 0) { // Verifica se a thread filha retornou um valor diferente de zero
            break; // Interrompe o loop se a conta 'from' zerou
        }
    }

    // Libera a pilha
    free(stack);
    printf("Todas as transferências foram concluídas e a memória foi liberada.\n");
    return 0;
}
