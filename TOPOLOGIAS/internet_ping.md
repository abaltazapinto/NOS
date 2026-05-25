

# Objectivo 

- Medir tres coisas basicas na tua ligacao por hotspot

	1. Latencia media - Tempo de resposta
	2. Jitter - variacao da latencia
	3. Packet loss - pacotes perdidos.

- Para IoT vais ver algo parecido com:
	120 packets transmitted, 118 received, 1.6% packet loss
	rtt min/avg/max/mdev = 45.2/92.4/230.1/35.7 ms

- O que interessa

| Métrica       |     Bom |   Problemático |
| ------------- | ------: | -------------: |
| Packet loss   |      0% |            >1% |
| Avg ping      |  <80 ms |        >150 ms |
| Max ping      | <250 ms |        >500 ms |
| mdev / jitter |   baixo | muito variável |


# Como pensar

O teu hotspot é uma cadeia assim:

	PC → Wi-Fi hotspot → telemóvel → rede móvel → Internet → servidor

Se o ping já tiver perdas ou latência muito instável, então qualquer IoT com SSH, MQTT, dashboards ou logs remotos vai sofrer.

- Para farm monitoring, a pergunta não é:

	“Tenho muitos Mbps?”

É:

	“A ligação aguenta pacotes pequenos de forma estável durante horas/dias?”



---

Deixa o comando de 120 rolar ate ao fim,

no fim aparece algo como 

--- 8.8.8.8 ping statistics ---
120 packets transmitted, 120 received, 0% packet loss
rtt min/avg/max/mdev = ...

a mim apareceu:

	--- 8.8.8.8 ping statistics ---	
	120 packets transmitted, 120 received, 0% packet loss, time 119612ms
	rtt min/avg/max/mdev = 265.789/1936.283/5593.521/1232.735 ms, pipe 6

# testes

## Ação — 1 passo 🔧

**Agora repete o mesmo teste, mas com o telemóvel junto a uma janela e o PC a menos de 1 metro do telemóvel:**

```bash
ping -c 120 8.8.8.8
```

## Objetivo

Separar duas hipóteses:

1. **Problema da rede móvel** — sinal fraco/congestionado.
2. **Problema do Wi-Fi hotspot** — PC longe do telemóvel ou interferência local.

## Interpretação do teu resultado

O teu teste foi:

```text
120 packets transmitted, 120 received, 0% packet loss
rtt min/avg/max/mdev = 265.789/1936.283/5593.521/1232.735 ms
```

### Leitura técnica

| Métrica       |     Valor | Interpretação              |
| ------------- | --------: | -------------------------- |
| Packet loss   |      `0%` | Bom: nenhum pacote perdido |
| RTT mínimo    |  `265 ms` | Alto                       |
| RTT médio     | `1936 ms` | Muito mau                  |
| RTT máximo    | `5593 ms` | Extremamente mau           |
| mdev / jitter | `1232 ms` | Muito instável             |

A ligação **não está a perder pacotes**, mas está a atrasá-los imenso.

Isto significa:

```text
A ligação existe, mas a latência é muito instável.
```

Para IoT:

| Uso                             | Resultado provável         |
| ------------------------------- | -------------------------- |
| Enviar dados MQTT a cada minuto | Pode funcionar             |
| Guardar logs remotamente        | Pode funcionar com retries |
| SSH remoto                      | Mau                        |
| Dashboard em tempo real         | Mau                        |
| Atualizações OTA                | Arriscado                  |
| Sistema industrial robusto      | Não aceitável sem fallback |

## Como pensar

O teu hotspot está a comportar-se como uma ligação com **bufferbloat, sinal fraco ou rede móvel congestionada**.

O ponto importante:

> Mbps não chega. Para IoT, precisas de previsibilidade.

Mesmo com `0% packet loss`, um `avg` de quase **2 segundos** e `max` de **5.6 segundos** indicam que os pacotes chegam, mas chegam tarde.

## Pitfalls

1. **Pensar que 0% loss = ligação boa**
   Não basta. A latência está péssima.

2. **Testar só uma vez**
   Rede móvel varia muito com hora, localização e carga da célula.

3. **Ignorar energia do telemóvel**
   Hotspot + rede móvel + ecrã pode aquecer e degradar.

4. **Usar SSH como critério principal**
   SSH sofre muito com jitter. MQTT tolera melhor se for bem configurado.

## Alternativas / tradeoffs

**Opção A — Hotspot telemóvel**
Boa para testes rápidos. Má para infraestrutura permanente.

**Opção B — Router 4G/5G dedicado com antena melhor**
Melhor para IoT real. Permite posicionamento, alimentação estável e antenas externas.

**Opção C — Wi-Fi do vizinho como principal + SIM como fallback**
Mais robusto, mas exige mais engenharia: watchdog, reconexão, failover e logs.

## Pergunta de decisão

No teste que fizeste, o telemóvel estava ligado por **4G/5G** ou estava a partilhar Internet recebida por **Wi-Fi da NOS**?

---

## Ação — 1 passo 🔧

**Faz um terceiro teste com o telemóvel na rua/exterior, mas mantém o PC perto dele, e corre novamente:**

```bash
ping -c 120 8.8.8.8
```

## Objetivo

Perceber se a barreira física da casa/janela está a degradar o sinal móvel.

Comparação dos teus dois testes:

| Local do telemóvel | Packet loss | RTT médio | RTT máximo | Jitter/mdev |
| ------------------ | ----------: | --------: | ---------: | ----------: |
| Antes              |          0% |   1936 ms |    5593 ms |     1232 ms |
| Janela             |          0% |   1660 ms |    5037 ms |      990 ms |

Melhorou, mas **ainda está mau**.

A janela reduziu a média e o jitter, logo a posição influencia. Mas um RTT médio de **1.66 segundos** continua demasiado alto para ligação interativa.

## Como pensar

Isto indica uma destas hipóteses:

```text
sinal móvel fraco
ou
célula móvel congestionada
ou
hotspot/telemóvel a degradar
ou
operador com rota má até 8.8.8.8
```

A janela ajudou, mas não resolveu. Portanto, antes de decidires “fico sempre na janela”, testa o exterior.

Para IoT:

* **sensores a enviar dados espaçados**: ainda pode funcionar;
* **SSH remoto**: mau;
* **dashboard em tempo real**: mau;
* **sistema robusto**: precisas de melhor ligação ou arquitetura tolerante a atrasos.

## Pitfalls

1. **Deixar o telemóvel sempre na rua**
   Não é solução estável: chuva, calor, bateria, roubo, humidade.

2. **Assumir que janela = melhor sempre**
   Às vezes outra divisão ou outra orientação apanha melhor antena.

3. **Confundir 0% packet loss com boa rede**
   Aqui o problema é latência extrema, não perda.

4. **Usar telemóvel como router permanente**
   Serve para teste, mas não é boa infraestrutura IoT.

## Alternativas / tradeoffs

**Opção A — Telemóvel na janela**
Melhor que dentro de casa, mas ainda instável.

**Opção B — Router 4G/5G dedicado perto da janela**
Mais realista para uso contínuo.

**Opção C — Antena exterior + router 4G/5G**
Melhor solução técnica se a cobertura for fraca.

**Opção D — Outro operador/SIM**
Pode ser a melhor solução se a célula Digi/NOS/Vodafone local estiver saturada.

## Pergunta de decisão

Consegues fazer esse terceiro teste **na rua/exterior** e colar só o resumo final?

---

## Ação — 1 passo 🔧

**Agora faz `traceroute` para veres onde começa a latência alta:**

```bash
traceroute 8.8.8.8
```

Se não tiveres instalado:

```bash
sudo apt install traceroute
```

## Objetivo

O teu resultado ao gateway foi excelente:

```text
rtt min/avg/max/mdev = 0.482/1.016/1.645/0.209 ms
```

Isto prova que:

```text
PC → Wi-Fi hotspot → telemóvel = estável
```

Portanto o problema **não está no Wi-Fi entre o PC e o telemóvel**.

O problema está depois:

```text
telemóvel → rede móvel → operador → Internet
```

## Como pensar

Temos agora duas medições:

| Alvo                           |  Média | Interpretação                    |
| ------------------------------ | -----: | -------------------------------- |
| Gateway hotspot `172.23.112.1` |  ~1 ms | Local está perfeito              |
| Google DNS `8.8.8.8`           | ~1.7 s | Rede móvel/operador está péssima |

Isto é um diagnóstico forte: o hotspot local funciona bem, mas a ligação móvel está com latência absurda.

## Pitfalls

1. **`traceroute` pode mostrar `* * *`**
   Alguns routers bloqueiam respostas ICMP. Não significa necessariamente falha.

2. **Operadores móveis usam CG-NAT**
   O primeiro salto externo pode não ser claro.

3. **Rede móvel varia por hora**
   Testa também manhã/tarde/noite.

4. **Router 4G/5G não resolve tudo**
   Se a célula da operadora estiver congestionada, hardware melhor ajuda pouco.

## Alternativas / tradeoffs

**Abordagem A — trocar posição do telemóvel**
Já testaste. Não melhorou o suficiente.

**Abordagem B — testar outro SIM/operador**
Provavelmente o próximo teste mais útil.

**Abordagem C — router 4G/5G com antena externa**
Só vale a pena se o problema for sinal fraco, não congestionamento da célula.

## Pergunta de decisão

Qual é o operador móvel do telemóvel neste teste: **NOS, Vodafone, MEO ou Digi**?

---

	traceroute 8.8.8.8.
Processing triggers for man-db (2.12.0-4build2) ...
abaltaza@DESKTOP-NET1NBD:~$ traceroute 8.8.8.8
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
 1  DESKTOP-NET1NBD.mshome.net (172.23.112.1)  0.538 ms  0.498 ms  0.516 ms
 2  10.177.17.246 (10.177.17.246)  3627.746 ms  3621.484 ms  3631.151 ms
 3  * * *
 4  * 172.16.18.161 (172.16.18.161)  3850.944 ms *
 5  10.130.6.155 (10.130.6.155)  3850.869 ms 10.130.6.153 (10.130.6.153)  3850.846 ms 10.130.6.155 (10.130.6.155)  3850.808 ms
 6  10.130.6.152 (10.130.6.152)  3850.970 ms  3874.165 ms  3847.323 ms
 7  10.130.35.103 (10.130.35.103)  3847.372 ms  4748.507 ms 10.130.12.9 (10.130.12.9)  3529.645 ms
 8  10.130.35.103 (10.130.35.103)  419.092 ms  409.641 ms  407.498 ms
 9  10.130.35.106 (10.130.35.106)  407.501 ms  407.421 ms  407.354 ms
10  10.255.48.74 (10.255.48.74)  426.768 ms 10.255.48.78 (10.255.48.78)  423.824 ms  423.760 ms
11  * * *
12  * * *
13  dns.google (8.8.8.8)  466.368 ms  511.191 ms  510.872 ms
abaltaza@DESKTOP-NET1NBD:~$

# dados

## Ação — 1 passo 🔧

**Faz agora o mesmo `ping`, mas para o primeiro salto da NOS depois do telemóvel:**

```bash
ping -c 60 10.177.17.246
```

## Objetivo

Confirmar se a latência absurda começa logo no primeiro nó da rede móvel NOS.

O teu `traceroute` mostra isto:

```text
1  172.23.112.1       ~0.5 ms
2  10.177.17.246      ~3620 ms
...
13 8.8.8.8            ~466–511 ms
```

Isto é muito interessante.

### Interpretação

| Hop | IP                     | Latência | Significado                                           |
| --- | ---------------------- | -------: | ----------------------------------------------------- |
| 1   | `172.23.112.1`         |  ~0.5 ms | PC → telemóvel está excelente                         |
| 2   | `10.177.17.246`        | ~3620 ms | Entrada na rede móvel NOS está péssima                |
| 3+  | `* * *` / IPs privados | variável | Core/NAT interno da NOS                               |
| 13  | `8.8.8.8`              |  ~500 ms | Google responde muito mais baixo que hops intermédios |

## Como pensar

A tua rede local está limpa:

```text
PC → Wi-Fi hotspot → telemóvel = OK
```

O problema começa aqui:

```text
telemóvel → torre/rede móvel NOS = mau
```

Mas atenção: em `traceroute`, routers intermédios podem responder devagar a ICMP de propósito. Por isso não basta olhar para o hop 2 e concluir imediatamente. Temos de testar diretamente esse hop com `ping`.

Se o comando:

```bash
ping -c 60 10.177.17.246
```

também der média acima de centenas ou milhares de ms, então a NOS móvel nesse local está muito má para uso interativo.

## O que isto sugere para o teu projeto IoT

Para **IoT tolerante a atraso**, ainda podes usar:

```text
sensor → buffer local → envio periódico → retries
```

Mas para:

* SSH remoto;
* dashboard live;
* controlo em tempo real;
* atualizações OTA frequentes;

esta ligação NOS hotspot é fraca.

## Pitfalls

1. **Routers internos podem despriorizar ping/traceroute**
   Nem sempre o hop lento é o gargalo real.

2. **IP `10.x.x.x` é privado**
   A NOS usa NAT/CG-NAT. Isto dificulta acesso remoto direto.

3. **Latência móvel varia com hora**
   Tens de testar em pelo menos 2 horários diferentes.

4. **Speedtest pode esconder este problema**
   Pode mostrar Mbps aceitáveis mas latência péssima.

## Alternativas / tradeoffs

**Opção A — manter NOS hotspot**
Aceitável só para testes e envio assíncrono de dados.

**Opção B — testar SIM Digi/Vodafone/MEO no mesmo local**
Melhor próximo diagnóstico para escolher operador.

**Opção C — router 4G/5G dedicado com antena externa**
Só faz sentido depois de descobrir qual operador tem melhor rede aí.

## Pergunta de decisão

O `ping -c 60 10.177.17.246` dá uma média parecida com milhares de ms ou fica baixa?

---

## Ação — 1 passo 🔧

**Agora faz ping diretamente ao destino final, mas com intervalo curto para veres jitter em tempo real:**

```bash
ping -i 0.2 -c 100 8.8.8.8
```

## Objetivo

Confirmar uma coisa importante: o `traceroute` mostrou o hop 2 com ~3600 ms, mas o `ping` direto a esse hop deu:

```text
rtt min/avg/max/mdev = 2.019/46.371/205.502/65.127 ms
```

Isto muda a interpretação.

O primeiro nó da NOS **não está permanentemente mau**. O valor absurdo no `traceroute` pode ter sido causado por:

```text
despriorização de ICMP/traceroute
ou
fila temporária/bufferbloat
ou
variação momentânea da rede móvel
```

## Como pensar

Temos agora:

| Teste                |                      Resultado | Interpretação          |
| -------------------- | -----------------------------: | ---------------------- |
| PC → telemóvel       |                          ~1 ms | Excelente              |
| PC → primeiro nó NOS |                   ~46 ms média | Aceitável              |
| PC → 8.8.8.8         |             ~1.7 s média antes | Muito mau              |
| traceroute           | valores altos em hops internos | Não conclusivo sozinho |

Isto sugere que o problema pode não estar logo na antena móvel. Pode estar mais à frente, no core/NAT/routing da NOS, ou ser **bufferbloat** quando a ligação está ocupada.

## O que observar no próximo teste

No novo `ping -i 0.2`, olha para os valores `time=`:

```text
time=40 ms
time=55 ms
time=80 ms
time=1500 ms
time=3000 ms
time=60 ms
```

Se vires muitos saltos para milhares de ms, a ligação está com jitter extremo.

Se ficar maioritariamente abaixo de 150–250 ms, então o teste anterior apanhou uma fase má.

## Pitfalls

1. **`traceroute` engana facilmente**
   Routers intermédios podem responder tarde a pacotes ICMP.

2. **8.8.8.8 pode não ser o melhor alvo**
   Depois podemos testar `1.1.1.1` e servidores NOS.

3. **Rede móvel oscila por segundos**
   Um teste isolado não chega.

4. **Hotspot pode ter tráfego em background**
   Updates, cloud sync ou apps do telemóvel podem criar filas.

## Alternativas / tradeoffs

**Abordagem A — repetir ping rápido para 8.8.8.8**
Boa para observar jitter.

**Abordagem B — testar `1.1.1.1`**
Ajuda a perceber se é rota específica para Google.

**Abordagem C — testar durante download/upload**
Diagnostica bufferbloat, mas só depois de termos baseline.

## Pergunta de decisão

Quando correres esse comando, o `avg` fica mais perto de **50–150 ms** ou continua acima de **1000 ms**?

---

## Ação — 1 passo 🔧

**Testa outro destino para separar “problema geral da NOS” de “rota má até Google”:**

```bash
ping -i 0.2 -c 100 1.1.1.1
```

## Objetivo

Comparar a rota NOS → Google DNS com NOS → Cloudflare DNS.

O teu resultado para `8.8.8.8` é péssimo:

```text
rtt min/avg/max/mdev = 769.181/2791.151/5439.359/1323.148 ms
```

Isto significa:

| Métrica |   Valor | Leitura                       |
| ------- | ------: | ----------------------------- |
| Perda   |      0% | Os pacotes chegam             |
| Mínimo  |  769 ms | Mesmo no melhor caso está mau |
| Média   | 2791 ms | Muito mau                     |
| Máximo  | 5439 ms | Quase inutilizável            |
| Jitter  | 1323 ms | Extremamente instável         |

Mas o ping ao primeiro nó da NOS foi razoável:

```text
rtt avg = 46 ms
```

Logo, o problema parece estar **depois da entrada na rede NOS**, provavelmente em routing, NAT/core móvel, congestionamento ou caminho específico até Google.

## Como pensar

Neste momento tens esta cadeia:

```text
PC → hotspot → telemóvel → NOS móvel → Internet
```

Já validaste:

```text
PC → hotspot → telemóvel = excelente
```

E validaste:

```text
PC → primeiro nó NOS = aceitável
```

Agora falta perceber se isto é:

```text
NOS → Google mau
```

ou:

```text
NOS → Internet em geral mau
```

Se `1.1.1.1` também der média acima de 1000 ms, a ligação NOS móvel está má para este local.
Se `1.1.1.1` der valores bons, então o problema pode ser rota específica para `8.8.8.8`.

## Pitfalls

1. **DNS não está a ser testado**
   `8.8.8.8` e `1.1.1.1` aqui são só IPs de destino; não estás a testar resolução DNS.

2. **Rede móvel oscila muito**
   Resultados devem ser comparados no mesmo local e na mesma hora.

3. **`traceroute` pode enganar**
   O `ping` final é mais útil para medir experiência real.

4. **Hotspot não é infraestrutura**
   Mesmo que melhore, telemóvel como router permanente é frágil.

## Alternativas / tradeoffs

**Opção A — Testar Cloudflare `1.1.1.1`**
Melhor próximo teste técnico.

**Opção B — Testar outro SIM no mesmo local**
Mais útil para escolher operador.

**Opção C — Usar router 4G/5G dedicado**
Só vale a pena depois de escolher o operador com melhor latência.

## Pergunta de decisão

O resultado para `1.1.1.1` fica parecido com o `8.8.8.8`, ou melhora muito?

---

## Ação — 1 passo 🔧

**Repete o mesmo teste para um servidor dentro da própria NOS:**

```bash
ping -i 0.2 -c 100 212.113.188.1
```

## Objetivo

Perceber se o problema é:

```text
NOS móvel → Internet em geral
```

ou se também acontece dentro da própria rede NOS.

O teu teste para Cloudflare confirmou que **não é só rota para Google**:

| Destino   | Perda | RTT médio | RTT máximo |  Jitter |
| --------- | ----: | --------: | ---------: | ------: |
| `8.8.8.8` |    0% |   2791 ms |    5439 ms | 1323 ms |
| `1.1.1.1` |    0% |   2588 ms |    6091 ms | 1660 ms |

Isto é mau para ligação interativa. A rede entrega pacotes, mas com filas enormes.

## Como pensar

A comparação importante até agora é esta:

```text
PC → hotspot → telemóvel: ~1 ms
PC → primeiro nó NOS: ~46 ms
PC → Internet pública: ~2600–2800 ms
```

Isto aponta para problema depois da entrada na rede móvel NOS: **core móvel, CG-NAT, routing, congestionamento ou filas/bufferbloat**.

Se o ping para `212.113.188.1` também der médias acima de 1000 ms, eu classificaria esta ligação NOS hotspot como **não adequada para SSH remoto nem monitorização em tempo quase real**.

## Pitfalls

1. **IP interno/externo pode bloquear ICMP**
   Se não responder, não quer dizer necessariamente que está em baixo.

2. **Rede móvel varia com hora**
   Convém repetir depois em outro horário.

3. **0% packet loss engana**
   Para IoT, pacotes atrasados 3–6 segundos também são problema.

4. **Router 4G pode não resolver routing mau**
   Se o problema for congestão da NOS, hardware melhor não corrige tudo.

## Alternativas / tradeoffs

**Opção A — Continuar com NOS**
Aceitável para sensores com envio espaçado, buffer local e retries.

**Opção B — Testar SIM Digi/Vodafone/MEO no mesmo local**
Melhor decisão custo/qualidade.

**Opção C — Usar Wi-Fi do vizinho + SIM como fallback**
Provavelmente a arquitetura mais robusta para o teu projeto.

## Pergunta de decisão

O ping para `212.113.188.1` fica também acima de **1000 ms**, ou baixa para valores tipo **50–150 ms**?

---

## Ação — 1 passo 🔧

**Ignora esse IP `212.113.188.1` e faz agora o teste para um domínio real da NOS:**

```bash
ping -i 0.2 -c 100 nos.pt
```

## Objetivo

O IP `212.113.188.1` provavelmente **não responde a ICMP/ping**.
O resultado:

```text
100 packets transmitted, 0 received, 100% packet loss
```

não prova que a ligação esteja morta. Só prova que **aquele IP não respondeu ao ping**.

O teste útil agora é ver se a latência também fica má para um destino associado à NOS.

## Como pensar

Até agora, tens isto:

| Teste                           |    Resultado | Interpretação                |
| ------------------------------- | -----------: | ---------------------------- |
| Hotspot local `172.23.112.1`    |        ~1 ms | PC ↔ telemóvel excelente     |
| Primeiro nó NOS `10.177.17.246` |   ~46 ms avg | Entrada na NOS aceitável     |
| Google `8.8.8.8`                | ~2791 ms avg | Muito mau                    |
| Cloudflare `1.1.1.1`            | ~2588 ms avg | Muito mau                    |
| `212.113.188.1`                 |    100% loss | IP não útil para diagnóstico |

Isto sugere que a ligação móvel NOS está a sofrer **depois dos primeiros hops**, ou que há algum comportamento estranho de routing/filas.

## Pitfalls

1. **Nem todo IP responde a ping**
   Firewalls podem bloquear ICMP.

2. **`ping nos.pt` testa DNS + rede**
   Como usas domínio, se demorar a arrancar pode haver atraso de DNS também.

3. **O site pode estar atrás de CDN**
   Pode não ir para “dentro da NOS” pura.

4. **Rede móvel pode estar congestionada**
   Se `nos.pt` também der >1000 ms, a conclusão fica forte.

## Alternativas / tradeoffs

**Abordagem A — ping a domínio NOS**
Boa para comparar com Google/Cloudflare.

**Abordagem B — testar outro operador no mesmo local**
Provavelmente a decisão prática mais importante.

**Abordagem C — usar Wi-Fi fixo/PTP para principal + SIM como fallback**
Mais robusto para o teu projeto IoT.

## Pergunta de decisão

O `ping -i 0.2 -c 100 nos.pt` dá média acima de **1000 ms** ou fica perto de **50–150 ms**?

---

## Ação — 1 passo 🔧

**Sim, podes reclamar à NOS — mas reclama por “latência/jitter extremo na Internet móvel”, não apenas por “velocidade”.**

Leva estes resultados numa nota:

```text
Hotspot local:
172.23.112.1
60 packets transmitted, 60 received, 0% packet loss
rtt min/avg/max/mdev = 0.482/1.016/1.645/0.209 ms

Primeiro nó NOS:
10.177.17.246
60 packets transmitted, 60 received, 0% packet loss
rtt min/avg/max/mdev = 2.019/46.371/205.502/65.127 ms

Google:
8.8.8.8
100 packets transmitted, 100 received, 0% packet loss
rtt min/avg/max/mdev = 769.181/2791.151/5439.359/1323.148 ms

Cloudflare:
1.1.1.1
100 packets transmitted, 100 received, 0% packet loss
rtt min/avg/max/mdev = 77.020/2588.405/6091.649/1660.783 ms

NOS site:
nos.pt
100 packets transmitted, 100 received, 0% packet loss
rtt min/avg/max/mdev = 508.203/2981.134/6507.877/1595.429 ms
```

## Objetivo

A tua evidência mostra que:

```text
PC → hotspot → telemóvel = bom
entrada inicial na NOS = aceitável
NOS móvel → Internet pública = muito instável
```

Isto é material suficiente para abrir ocorrência técnica. A ANACOM diz que os operadores não têm necessariamente mínimos legais gerais de qualidade iguais para todos os serviços, mas os contratos devem indicar condições de qualidade e níveis aplicáveis; por isso, o melhor é pedir à NOS que verifique a qualidade real do serviço na tua zona/localização. ([Anacom Consumer Portal][1])

## Como reclamar melhor

Diz algo deste género:

> “Estou a usar Internet móvel NOS por hotspot. A rede local entre o PC e o telemóvel está estável, com cerca de 1 ms para o gateway. Mas para destinos externos como 8.8.8.8, 1.1.1.1 e nos.pt tenho médias entre 2.5 e 3 segundos, com picos acima de 6 segundos, sem packet loss. Peço verificação técnica da rede móvel nesta zona, porque a latência e jitter tornam o serviço inadequado para uso normal.”

Não digas só:

> “A Internet está lenta.”

Isso é vago. Diz:

> “Tenho latência média de 2.5–3 s e jitter superior a 1.5 s em múltiplos destinos.”

## Pitfalls

1. **Eles podem pedir Speedtest**
   Faz também 2 ou 3 Speedtests, mas guarda os pings. Speedtest sozinho não mostra bem jitter.

2. **Podem culpar o hotspot**
   O teu ping para `172.23.112.1` refuta isso: PC → telemóvel está ótimo.

3. **Podem dizer que rede móvel não garante latência**
   Mesmo assim, podes pedir verificação de cobertura/congestionamento.

4. **Loja pode não resolver**
   Melhor abrir ocorrência técnica por apoio ao cliente e guardar número do processo.

## Alternativas / tradeoffs

**Opção A — Reclamar à NOS primeiro**
Boa se já pagas o serviço e queres tentar correção.

**Opção B — Testar Digi/Vodafone/MEO no mesmo local**
Melhor decisão prática para o teu projeto IoT.

**Opção C — Reclamação formal**
Podes usar o Livro de Reclamações físico ou eletrónico; segundo a ANACOM, a reclamação eletrónica tem a mesma validade da física, e o operador deve responder no prazo máximo de 15 dias úteis. ([Anacom Consumer Portal][2])

## Pergunta de decisão

Queres ir primeiro pela via **apoio técnico NOS** ou queres preparar já uma **reclamação formal curta**?

[1]: https://www.anacom-consumidor.pt/faq-qualidade-do-servico?utm_source=chatgpt.com "Qualidade do serviço - Portal do Consumidor"
[2]: https://www.anacom-consumidor.pt/livro-reclamacoes?utm_source=chatgpt.com "Livro de reclamações - Portal do Consumidor"

---

# reclamacao importante 



Hotspot local:
172.23.112.1
60 packets transmitted, 60 received, 0% packet loss
rtt min/avg/max/mdev = 0.482/1.016/1.645/0.209 ms

Primeiro nó NOS:
10.177.17.246
60 packets transmitted, 60 received, 0% packet loss
rtt min/avg/max/mdev = 2.019/46.371/205.502/65.127 ms

Google:
8.8.8.8
100 packets transmitted, 100 received, 0% packet loss
rtt min/avg/max/mdev = 769.181/2791.151/5439.359/1323.148 ms

Cloudflare:
1.1.1.1
100 packets transmitted, 100 received, 0% packet loss
rtt min/avg/max/mdev = 77.020/2588.405/6091.649/1660.783 ms

NOS site:
nos.pt
100 packets transmitted, 100 received, 0% packet loss
rtt min/avg/max/mdev = 508.203/2981.134/6507.877/1595.429 ms


evidencia mostra que:

	PC → hotspot → telemóvel = bom
	entrada inicial na NOS = aceitável
	NOS móvel → Internet pública = muito instável

Isto é material suficiente para abrir ocorrência técnica. A ANACOM diz que os operadores não têm necessariamente mínimos legais gerais de qualidade iguais para todos os serviços, mas os contratos devem indicar condições de qualidade e níveis aplicáveis; por isso, o melhor é pedir à NOS que verifique a qualidade real do serviço na tua zona/localização.


---

## Ação — 1 passo 🔧

**Sim, podes reclamar à NOS — mas reclama por “latência/jitter extremo na Internet móvel”, não apenas por “velocidade”.**

Leva estes resultados numa nota:

```text
Hotspot local:
172.23.112.1
60 packets transmitted, 60 received, 0% packet loss
rtt min/avg/max/mdev = 0.482/1.016/1.645/0.209 ms

Primeiro nó NOS:
10.177.17.246
60 packets transmitted, 60 received, 0% packet loss
rtt min/avg/max/mdev = 2.019/46.371/205.502/65.127 ms

Google:
8.8.8.8
100 packets transmitted, 100 received, 0% packet loss
rtt min/avg/max/mdev = 769.181/2791.151/5439.359/1323.148 ms

Cloudflare:
1.1.1.1
100 packets transmitted, 100 received, 0% packet loss
rtt min/avg/max/mdev = 77.020/2588.405/6091.649/1660.783 ms

NOS site:
nos.pt
100 packets transmitted, 100 received, 0% packet loss
rtt min/avg/max/mdev = 508.203/2981.134/6507.877/1595.429 ms
```

## Objetivo

A tua evidência mostra que:

```text
PC → hotspot → telemóvel = bom
entrada inicial na NOS = aceitável
NOS móvel → Internet pública = muito instável
```

Isto é material suficiente para abrir ocorrência técnica. A ANACOM diz que os operadores não têm necessariamente mínimos legais gerais de qualidade iguais para todos os serviços, mas os contratos devem indicar condições de qualidade e níveis aplicáveis; por isso, o melhor é pedir à NOS que verifique a qualidade real do serviço na tua zona/localização. ([Anacom Consumer Portal][1])

## Como reclamar melhor

Diz algo deste género:

> “Estou a usar Internet móvel NOS por hotspot. A rede local entre o PC e o telemóvel está estável, com cerca de 1 ms para o gateway. Mas para destinos externos como 8.8.8.8, 1.1.1.1 e nos.pt tenho médias entre 2.5 e 3 segundos, com picos acima de 6 segundos, sem packet loss. Peço verificação técnica da rede móvel nesta zona, porque a latência e jitter tornam o serviço inadequado para uso normal.”

Não digas só:

> “A Internet está lenta.”

Isso é vago. Diz:

> “Tenho latência média de 2.5–3 s e jitter superior a 1.5 s em múltiplos destinos.”

## Pitfalls

1. **Eles podem pedir Speedtest**
   Faz também 2 ou 3 Speedtests, mas guarda os pings. Speedtest sozinho não mostra bem jitter.

2. **Podem culpar o hotspot**
   O teu ping para `172.23.112.1` refuta isso: PC → telemóvel está ótimo.

3. **Podem dizer que rede móvel não garante latência**
   Mesmo assim, podes pedir verificação de cobertura/congestionamento.

4. **Loja pode não resolver**
   Melhor abrir ocorrência técnica por apoio ao cliente e guardar número do processo.

## Alternativas / tradeoffs

**Opção A — Reclamar à NOS primeiro**
Boa se já pagas o serviço e queres tentar correção.

**Opção B — Testar Digi/Vodafone/MEO no mesmo local**
Melhor decisão prática para o teu projeto IoT.

**Opção C — Reclamação formal**
Podes usar o Livro de Reclamações físico ou eletrónico; segundo a ANACOM, a reclamação eletrónica tem a mesma validade da física, e o operador deve responder no prazo máximo de 15 dias úteis. ([Anacom Consumer Portal][2])

## Pergunta de decisão

Queres ir primeiro pela via **apoio técnico NOS** ou queres preparar já uma **reclamação formal curta**?

[1]: https://www.anacom-consumidor.pt/faq-qualidade-do-servico?utm_source=chatgpt.com "Qualidade do serviço - Portal do Consumidor"
[2]: https://www.anacom-consumidor.pt/livro-reclamacoes?utm_source=chatgpt.com "Livro de reclamações - Portal do Consumidor"

---

