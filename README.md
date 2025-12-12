# POO---ARQUITETURA_DO_SISTEMA]
### **TRABALHO PRÁTICO \- PROGRAMAÇÃO CONCORRENTE**

### **MATEUS HENRIQUE DE SOUZA PINHEIRO**

### **MATRÍCULA: 2023010576**  **3\. Entregáveis e Critérios de Avaliação O trabalho deve ser entregue em um repositório Git contendo:** 

### **1\. Código Fonte: Classes Onibus, AgenteVenda, Cancelamento e Main.** 

### **2\. Relatório (README):** 

### **○ Print do console mostrando o erro de Overbooking (Parte I).** 

![][image1]

### **○ Explicação de como o synchronized resolveu a inconsistência de dados.** 

O `synchronized` interferiu no fluxo de execução das threads, garantindo que a operação de venda fosse **atômica** (indivisível), corrigindo o problema da **Race Condition** (Condição de Corrida) da Parte I.

## **1\. O Problema da Inconsistência (Parte I)**

O erro de *overbooking* (saldo negativo) ocorreu porque a operação de reserva era vulnerável a interrupções:

1. **Verificação:** O Agente A verificava se `assentosDisponiveis > 0`.  
2. **Latência:** A thread dormia (`Thread.sleep(100)`), simulando latência de rede.  
3. **Interrupção:** Durante a latência, o Agente B verificava e encontrava a mesma condição como verdadeira.  
4. **Venda Duplicada:** Ambos os agentes continuavam a execução e decrementavam o contador, resultando em um valor final inconsistente (negativo).

## **2\. A Solução do `synchronized` (Parte II)**

A solução foi aplicar o **Bloco Sincronizado** (`synchronized (this)`) exatamente ao redor da seção crítica do código: a verificação (`if`) e o decremento do assento.

### **Mecanismo de Exclusão Mútua**

O `synchronized` resolveu a inconsistência da seguinte forma:

1. **Aquisição do Lock:** Quando o **Agente A** (thread) começa a executar o código dentro do bloco `synchronized (this)`, ele automaticamente adquire o **lock** (o "cadeado") na instância do objeto `Onibus` (`this`).  
2. **Bloqueio de Threads:** Qualquer outro agente (por exemplo, **Agente B**) que tentar entrar nesse mesmo bloco sincronizado é **bloqueado**. O Agente B é forçado a esperar até que o lock seja liberado.  
3. **Garantia de Atomicidade:** Como o Agente A mantém o lock, ele pode executar o bloco inteiro, incluindo a `Thread.sleep(100)` e a linha `assentosDisponiveis--`, sem ser interrompido por outras threads tentando executar as mesmas linhas.  
4. **Liberação e Consistência:** Somente após o Agente A finalizar o decremento (e o contador, por exemplo, ir para 0), ele libera o lock. O Agente B, então, adquire o lock e entra no bloco. Ao fazer a checagem (`if (assentosDisponiveis > 0)`), ele vê o valor **atualizado e consistente** (0) e é impedido de realizar a venda, **eliminando o *overbooking***.

### **○ Explicação de como o wait/notify economiza CPU evitando busy-waiting**

O uso dos métodos **`wait()`** e **`notify()`** economiza o tempo de processamento da CPU ao mudar o estado de *threads* que estão em espera, impedindo o padrão ineficiente de **busy-waiting**.

**O Gasto do Busy-Waiting**

O **busy-waiting** (espera ocupada) é o método ineficiente de aguardar uma condição. Uma *thread* que usa *busy-waiting* fica em um *loop* (`while`) checando repetidamente uma variável compartilhada.

* **Estado:** A *thread* permanece nos estados **`Runnable`** ou **`Running`**.  
* **Custo:** O **processador aloca tempo** para essa *thread* executar a instrução de checagem, mesmo que a condição seja falsa. Esse consumo de ciclos de CPU é um **desperdício**, pois a *thread* está usando poder computacional em uma atividade inútil de espera, competindo por recursos com *threads* produtivas.

## **A Eficiência de `wait()` e `notify()`**

`wait()` e `notify()` fornecem um mecanismo de **comunicação por eventos** que força a *thread* a entrar em um estado passivo, eliminando o consumo de CPU.

### **Mecanismo de Suspensão**

1. **Liberação do Lock e Mudança de Estado:** Quando uma *thread* encontra a condição insatisfatória (ônibus lotado, por exemplo) e chama **`wait()`**, ela faz duas ações cruciais:  
   * **Libera o lock** que estava segurando no bloco sincronizado.  
   * Muda seu estado para **`Waiting`** (Esperando) ou **`Blocked`**.  
2. **Economia de CPU:** Ao entrar no estado `Waiting`, a *thread* é **retirada do agendador** (*scheduler*) do sistema operacional. Ela **não compete** mais por tempo de CPU e não é executada, o que resulta em **zero consumo** de ciclos de processamento enquanto espera.

### **Reativação Sob Demanda**

A *thread* só é reativada quando o evento esperado ocorre (o cancelamento libera um assento):

* O produtor (Cancelamento) chama **`notify()`** ou **`notifyAll()`**.  
* Isso sinaliza o *kernel* para mover a *thread* que estava em `Waiting` de volta para o estado `Blocked` (pronta para competir pelo *lock* e rodar).

Em resumo, em vez de **gastar CPU** verificando repetidamente (busy-waiting), a *thread* com `wait()` é **suspensa pelo Kernel** até receber um sinal específico, tornando a utilização do processador significativamente mais eficiente.

**O código utilizado para gerar a saída mostrada na imagem (print) envolveu modificações nos arquivos** `Onibus.java` e `Main.java`.

###  **Onibus. java** 

### **fazer a substituição**

import java.util.concurrent.Semaphore;

public class Onibus {  
    // Parte I: Assentos iniciais  
    private int assentosDisponiveis \= 5;  
      
    // Parte IV: Comentado para o CAOS  
    // private final Semaphore limiteConexao \= new Semaphore(3); 

    public void reservarAssento(String agente) {  
        // Parte IV: Comentado para o CAOS  
        /\*  
        try {  
            System.out.println("🚦 " \+ agente \+ " está aguardando permissão para CONEXÃO.");  
            limiteConexao.acquire();  
            System.out.println("🟢 " \+ agente \+ " adquiriu permissão de CONEXÃO. Tentando reservar assento.");  
        } catch (InterruptedException e) {  
            Thread.currentThread().interrupt();  
            return;  
        }  
        \*/

        // \*\*\* PARTE II e III: Comentado para o CAOS (Retire o 'synchronized' para forçar o erro) \*\*\*  
        // synchronized (this) {   
              
            // Parte III: Comentado para o CAOS (Retire o 'while' e 'wait()' para forçar o erro)  
            /\*  
            while (assentosDisponiveis \<= 0\) {  
                try {  
                    System.out.println("🛑 " \+ agente \+ ": Ônibus lotado. Aguardando cancelamento (wait()).");  
                    wait();  
                    System.out.println("🔔 " \+ agente \+ ": Fui notificado\! Tentando reservar novamente.");  
                } catch (InterruptedException e) {  
                    Thread.currentThread().interrupt();  
                    return;  
                }  
            }  
            \*/

            // \*\*\* Seção Crítica da Parte I (Onde ocorre a Race Condition) \*\*\*  
            if (assentosDisponiveis \> 0\) {  
                try {  
                    // Simula a latência de rede/sistema (essencial para a Race Condition)  
                    System.out.println(agente \+ " verifica: " \+ assentosDisponiveis \+ ". Dormindo 100ms...");  
                    Thread.sleep(100);   
                } catch (InterruptedException e) {  
                    Thread.currentThread().interrupt();  
                    return;  
                }

                assentosDisponiveis--;  
                System.out.println("✅ " \+ agente \+ " reservou\! Assentos restantes: " \+ assentosDisponiveis);  
            } else {  
                 System.out.println("❌ " \+ agente \+ " FALHOU na reserva (Ônibus lotado).");  
            }  
        // } // Fim do Bloco Synchronized Comentado

        // Parte IV: Comentado para o CAOS  
        /\*  
        finally {  
            limiteConexao.release();  
            System.out.println("🔓 " \+ agente \+ " liberou permissão de CONEXÃO.");  
        }  
        \*/  
    }

    // O método cancelarPassagem não será usado na Parte I, mas pode ficar aqui  
    public void cancelarPassagem(String agente) {  
        synchronized (this) {  
            assentosDisponiveis++;  
            System.out.println("\\n📣 " \+ agente \+ " CANCELOU uma passagem. Assentos restantes: " \+ assentosDisponiveis);  
            notifyAll();   
            System.out.println("📢 Notificando todos os agentes em espera...");  
        }  
    }

    public int getAssentosDisponiveis() {  
        return assentosDisponiveis;  
    }  
}

**Main.java**   
public class Main {

    public static void main(String\[\] args) throws InterruptedException {  
        Onibus amatur \= new Onibus();   
          
        System.out.println("=================================================");  
        System.out.println("          Parte I: O Caos (Race Condition)");  
        System.out.println("  (Assentos: 5\. Agentes: 7\. SEM Sincronização)");  
        System.out.println("=================================================");

        // Criando 7 Agentes de Venda  
        Thread\[\] agentes \= new Thread\[7\];  
        for (int i \= 0; i \< 7; i++) {  
            agentes\[i\] \= new Thread(new AgenteVenda(amatur, "Agente " \+ (i \+ 1)));  
        }

        // Não vamos iniciar a thread de Cancelamento para esta demonstração\!  
          
        // Iniciando todas as threads (Agentes)  
        for (Thread agente : agentes) {  
            agente.start();  
        }

        // Aguarda a conclusão de todas as threads  
        for (Thread agente : agentes) {  
            agente.join();  
        }

        System.out.println("\\n=================================================");  
        System.out.println("       FIM da Simulação");  
        System.out.println("       Assentos finais: " \+ amatur.getAssentosDisponiveis());  
        System.out.println("=================================================");  
    }  
}

[image1]: <data:image/png;base64,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>
