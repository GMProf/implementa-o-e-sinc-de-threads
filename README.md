# Códigos completos dos gabaritos

## Arquivo: SolucaoAtividade1.java

```java
public class SolucaoAtividade1 {
    private static void executarTarefa() {
        for (int i = 1; i <= 3; i++) {
            System.out.println(Thread.currentThread().getName()
                    + " executou item " + i);
        }
    }

    static class Trabalhadora extends Thread {
        Trabalhadora(String nome) { super(nome); }
        @Override
        public void run() { executarTarefa(); }
    }

    public static void main(String[] args)
            throws InterruptedException {
        Thread a = new Trabalhadora("heranca");
        Runnable tarefa = SolucaoAtividade1::executarTarefa;
        Thread b = new Thread(tarefa, "composicao");
        a.start();
        b.start();
        a.join();
        b.join();
        System.out.println("Finalizado apos as duas tarefas.");
    }
}
```

## Arquivo: SolucaoAtividade2.java

```java
public class SolucaoAtividade2 {
    static class Contador {
        private int total;
        synchronized void registrar() { total++; }
        synchronized int getTotal() { return total; }
    }

    public static void main(String[] args)
            throws InterruptedException {
        Contador contador = new Contador();
        Thread[] workers = new Thread[4];
        Runnable tarefa = () -> {
            for (int i = 0; i < 25_000; i++) {
                contador.registrar();
            }
        };
        // Iniciar todas antes de aguardar preserva a concorrencia.
        for (int i = 0; i < workers.length; i++) {
            workers[i] = new Thread(tarefa, "worker-" + i);
            workers[i].start();
        }
        for (Thread worker : workers) { worker.join(); }
        int obtido = contador.getTotal();
        if (obtido != 100_000) {
            throw new AssertionError("Total incorreto: " + obtido);
        }
        System.out.println("Registros consistentes: " + obtido);
    }
}
```

## Arquivo: SolucaoAtividade3.java

```java
public class SolucaoAtividade3 {
    static class Estoque {
        private final Object trava = new Object();
        private int disponivel;

        Estoque(int inicial) {
            if (inicial < 0) {
                throw new IllegalArgumentException("Estoque negativo.");
            }
            disponivel = inicial;
        }

        boolean reservar(int quantidade) {
            if (quantidade <= 0) {
                throw new IllegalArgumentException("Quantidade invalida.");
            }
            synchronized (trava) {
                // Verificar e alterar pertencem a mesma regiao critica.
                if (quantidade > disponivel) { return false; }
                disponivel -= quantidade;
                return true;
            }
        }

        int getDisponivel() {
            synchronized (trava) { return disponivel; }
        }
    }

    public static void main(String[] args)
            throws InterruptedException {
        Estoque estoque = new Estoque(100);
        Thread[] workers = new Thread[4];
        int[] sucessos = new int[workers.length];
        for (int i = 0; i < workers.length; i++) {
            final int indice = i;
            workers[i] = new Thread(() -> {
                for (int tentativa = 0; tentativa < 40; tentativa++) {
                    if (estoque.reservar(2)) { sucessos[indice]++; }
                }
            }, "vendas-" + i);
            workers[i].start();
        }
        for (Thread worker : workers) { worker.join(); }
        // Cada thread escreveu apenas em sua posicao do vetor.
        int reservas = 0;
        for (int valor : sucessos) { reservas += valor; }
        int saldo = estoque.getDisponivel();
        if (reservas != 50 || saldo != 0 || saldo + reservas * 2 != 100) {
            throw new AssertionError("Invariante de estoque violada.");
        }
        System.out.println("Reservas aceitas: " + reservas);
        System.out.println("Reservas recusadas: " + (160 - reservas));
        System.out.println("Unidades disponiveis: " + saldo);
    }
}
```

## Arquivo: TestesEstoque.java

```java
public class TestesEstoque {
    private static void exigir(boolean condicao, String mensagem) {
        if (!condicao) { throw new AssertionError(mensagem); }
    }

    public static void main(String[] args) {
        SolucaoAtividade3.Estoque e = new SolucaoAtividade3.Estoque(3);
        exigir(!e.reservar(4), "Reserva excessiva aceita.");
        exigir(e.getDisponivel() == 3, "Recusa alterou o saldo.");
        exigir(e.reservar(3), "Reserva exata recusada.");
        exigir(e.getDisponivel() == 0, "Saldo deveria ser zero.");
        exigir(!e.reservar(1), "Reserva em estoque vazio aceita.");
        for (int valor : new int[] {0, -1}) {
            try {
                e.reservar(valor);
                throw new AssertionError("Valor invalido aceito.");
            } catch (IllegalArgumentException esperado) {
                exigir(e.getDisponivel() == 0, "Falha alterou o saldo.");
            }
        }
        try {
            new SolucaoAtividade3.Estoque(-1);
            throw new AssertionError("Estoque negativo aceito.");
        } catch (IllegalArgumentException esperado) {
            System.out.println("Casos de limite do estoque aprovados.");
        }
    }
}
```
