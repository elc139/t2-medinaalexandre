# T2: Programação Paralela Multithread 
Alexandre Moreira Medina - ELC139

# Questões PThreads
1. Explique como se encontram implementadas as 4 etapas de projeto: particionamento, comunicação, aglomeração, mapeamento (use trechos de código para ilustrar a explicação).
    
    - Particionamento:
    Para poder utilizar paralelismo, o programa cria n threads, e dentro de cada uma dessas threads fica responsável por uma parte especifica do vetor.
     ```c
        // Criação das threads dentro da função dotprod_threads()
        for (i = 0; i < nthreads; i++) {
          pthread_create(&threads[i], &attr, dotprod_worker, (void *) i);
        }
        
        // na função dotprod_worker() acontece a distribuição e execução
        
        // atribuição da parte em que a thread vai ser responsável
           int wsize = dotdata.wsize;
           int start = offset*wsize;
           int end = start + wsize;
        
        // execução
           for (k = 0; k < dotdata.repeat; k++) {
              mysum = 0.0;
              for (i = start; i < end ; i++)  {
                 mysum += (a[i] * b[i]);
              }
           }
     ```
   
   - Comunicação: Para satisfazer as dependências, é utilizado a mutação exclusiva da variavel mysum, a mesma que foi utilizada na execução desmontrada no trecho de código acima.
    ```c
        pthread_mutex_lock (&mutexsum);
        dotdata.c += mysum;
        pthread_mutex_unlock (&mutexsum);
    ```
   - Aglomeração: Para reduzir o tempo de comunicação, primeiro é executada toda a tarefa para compartilhar o resultado com as demais threads
   ```c
       for (k = 0; k < dotdata.repeat; k++) {
         mysum = 0.0;
         for (i = start; i < end ; i++)  {
            mysum += (a[i] * b[i]);
         }
       }
   
       pthread_mutex_lock (&mutexsum);
       dotdata.c += mysum;
       pthread_mutex_unlock (&mutexsum);
   ```
   - Mapeamento: O balancemento da carga é feita de forma estática, dividindo em n threads definidas pelo usuário.
      ```c
          nthreads = atoi(argv[1]); 
          wsize = atoi(argv[2]);  // worksize = tamanho do vetor de cada thread
          .
          .
          .
          // na função dotprod_threads() 
          for (i = 0; i < nthreads; i++) {
            pthread_create(&threads[i], &attr, dotprod_worker, (void *) i);
          }
      ```
     
2. Considerando o tempo (em microssegundos) mostrado na saída do programa, qual foi a aceleração (speedup) com o uso de threads?

    1,98

3. A aceleração se sustenta para outros tamanhos de vetores, números de threads e repetições? Para responder a essa questão, você terá que realizar diversas execuções, variando o tamanho do problema (tamanho dos vetores e número de repetições) e o número de threads (1, 2, 4, 8..., dependendo do número de núcleos). Cada caso deve ser executado várias vezes, para depois calcular-se um tempo de processamento médio para cada caso. Atenção aos fatores que podem interferir na confiabilidade da medição: uso compartilhado do computador, tempos muito pequenos, etc.

    Sim
    
4. Elabore um gráfico/tabela de aceleração a partir dos dados obtidos no exercício anterior.

    | tool     | nthreads | size    | repetitions | usec    | 
    |----------|----------|---------|-------------|---------| 
    | Pthreads | 1        | 1000000 | 2000        | 6243630 | 
    | Pthreads | 2        | 500000  | 2000        | 3146861 | 
    | Pthreads | 4        | 250000  | 2000        | 1548837 | 
    | Pthreads | 8        | 125000  | 2000        | 1203875 | 
    | OpenMP   | 1        | 1000000 | 2000        |     -   | 
    | OpenMP   | 2        | 500000  | 2000        |      -  | 

5. Explique as diferenças entre pthreads_dotprod.c e pthreads_dotprod2.c. Com as linhas removidas, o programa está correto?

    O código do **pthreads_dotprod2.c** não faz o uso da exclusão mutua, podendo assim gerar um resultado errado caso duas threads utilizem a variavel **dotproc.c** ao mesmo tempo. Sem o uso da exclusão mutua o programa não está correto.
