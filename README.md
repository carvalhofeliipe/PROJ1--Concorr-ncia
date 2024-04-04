COMO COMPILAR 
Para compilar e executar o programa fornecido, você precisará de um ambiente de desenvolvimento C adequado, nós utilizamos o GCC.

$ gcc -o transferencia_program transferencia.c -lpthread

COMO EXECUTAR
$ ./transferencia_program

COMPROVAÇÃO DOS RESULTADOS
Deve-se observar a saída do programa. No console, será indicando se as transferências foram bem-sucedidas ou não, juntamente com os saldos das contas após cada transferência.

SOBRE O PROBLEMA
O problema de concorrência apresentado no código fornecido ocorre devido à falta de sincronização entre as threads que executam a função transferencia(). Para garantir a consistência dos saldos e evitar condições de corrida, é necessário utilizar mecanismos de sincronização, como semáforos, mutexes ou variáveis de condição. Para corrigir o problemas fizemos:

Utilização de Mutexes: Implementamos mutexes individuais para cada conta, garantindo que apenas uma thread possa acessar e modificar o saldo de cada conta por vez. Isso previne conflitos quando múltiplas threads tentam modificar os mesmos dados simultaneamente.

Ajustes na Função de Transferência: Modificamos a função de transferência para incluir o bloqueio e desbloqueio dos mutexes antes e depois das operações de atualização do saldo. Com isso, asseguramos que as operações de débito e crédito sejam realizadas de forma atômica, evitando inconsistências nos saldos.

Controle de Fluxo Aprimorado: Implementamos uma lógica para interromper as transferências caso o saldo da conta de origem não seja suficiente para completar a operação. Essa medida foi tomada para evitar saldos negativos e garantir a integridade dos dados.

Exibição de Resultados: Reorganizamos as instruções de impressão para que os saldos das contas sejam exibidos após cada operação de transferência. Dessa forma, facilitamos o acompanhamento do processo e a identificação de eventuais problemas.