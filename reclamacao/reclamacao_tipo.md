## Ação — 1 passo 🔧

**Copia e envia esta reclamação formal à NOS, idealmente pelo apoio ao cliente ou Livro de Reclamações Eletrónico.**

---

## Reclamação formal — Internet móvel NOS

**Assunto:** Reclamação por latência e jitter extremos em Internet móvel NOS

Exmos. Senhores,

Venho por este meio apresentar reclamação relativamente ao desempenho da minha ligação de Internet móvel NOS, utilizada através de hotspot móvel.

O problema principal não é apenas velocidade reduzida, mas sim **latência extremamente elevada e jitter muito instável**, tornando a ligação inadequada para utilização normal, acesso remoto, trabalho técnico, chamadas, navegação estável e monitorização IoT.

Foram realizados vários testes técnicos a partir de um computador ligado ao hotspot do telemóvel. Para excluir problema na ligação local entre o computador e o telemóvel, foi testado o gateway do hotspot:

```text
Ping ao gateway local do hotspot: 172.23.112.1
60 packets transmitted, 60 received, 0% packet loss
rtt min/avg/max/mdev = 0.482/1.016/1.645/0.209 ms
```

Este resultado demonstra que a ligação local entre o computador e o telemóvel está estável.

Também foi testado o primeiro nó da rede NOS:

```text
Ping ao primeiro nó NOS: 10.177.17.246
60 packets transmitted, 60 received, 0% packet loss
rtt min/avg/max/mdev = 2.019/46.371/205.502/65.127 ms
```

No entanto, ao testar destinos externos e o próprio domínio da NOS, os resultados mostram latência anormalmente elevada:

```text
Ping para 8.8.8.8
100 packets transmitted, 100 received, 0% packet loss
rtt min/avg/max/mdev = 769.181/2791.151/5439.359/1323.148 ms
```

```text
Ping para 1.1.1.1
100 packets transmitted, 100 received, 0% packet loss
rtt min/avg/max/mdev = 77.020/2588.405/6091.649/1660.783 ms
```

```text
Ping para nos.pt
100 packets transmitted, 100 received, 0% packet loss
rtt min/avg/max/mdev = 508.203/2981.134/6507.877/1595.429 ms
```

Estes valores indicam médias de latência entre aproximadamente **2,5 e 3 segundos**, com picos superiores a **6 segundos**, apesar de não haver perda de pacotes. Esta situação revela forte instabilidade, jitter extremo e possível congestionamento, routing deficiente ou anomalia na rede móvel NOS nesta zona.

Solicito, por isso:

1. Abertura de ocorrência técnica;
2. Verificação da qualidade da rede móvel NOS na minha localização;
3. Análise de latência, jitter, routing e eventual congestionamento;
4. Informação clara sobre se existe limitação, avaria, saturação de célula ou degradação do serviço;
5. Proposta de resolução ou alternativa, caso o serviço não consiga garantir condições mínimas de utilização estável.

Peço que esta reclamação seja registada formalmente e que me seja indicado o respetivo número de processo.

Com os melhores cumprimentos,
**André Pinto**
Contacto: __________________
N.º de cliente / contrato: __________________
Localização onde foram feitos os testes: __________________
Data: ___ / ___ / ______

---

## Objetivo

A reclamação está formulada tecnicamente para evitar que a NOS responda apenas com “faça um Speedtest”. O ponto forte é este:

> A rede local está boa, mas a latência para destinos externos é extremamente má.

## Como pensar

Não reclames só de “Internet lenta”. Isso é demasiado genérico.

O argumento técnico é:

```text
PC → hotspot → telemóvel = ~1 ms
PC → primeiro nó NOS = ~46 ms
PC → Internet pública / nos.pt = 2500–3000 ms
```

Isto mostra que o problema não está no teu PC nem no Wi-Fi do hotspot.

## Pitfalls

1. **A loja pode tentar simplificar o problema**
   Insiste em “latência e jitter”, não só “velocidade”.

2. **Podem pedir teste por Speedtest**
   Faz, mas não deixes que substitua os testes de ping.

3. **Podem culpar o telemóvel**
   Mostra o ping ao gateway local: ~1 ms.

4. **Podem dizer que rede móvel varia**
   Sim, mas 3 segundos de latência média é uma degradação forte.

## Pergunta de decisão

Queres que eu transforme isto numa versão **mais curta e agressiva para Livro de Reclamações**, com limite mais direto e linguagem menos técnica?
