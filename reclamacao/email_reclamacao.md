## Email rápido para a NOS

**Assunto:** Pedido de verificação técnica — latência extrema na Internet móvel NOS

Exmos. Senhores,

Venho solicitar a abertura de uma ocorrência técnica relativa à minha ligação de Internet móvel NOS, utilizada através de hotspot.

Após vários testes, verifiquei que a ligação entre o computador e o hotspot está estável, mas a ligação para a Internet apresenta latência extremamente elevada e jitter muito instável.

Resultados obtidos:

```text
Gateway local do hotspot:
60 packets transmitted, 60 received, 0% packet loss
rtt min/avg/max/mdev = 0.482/1.016/1.645/0.209 ms

Primeiro nó NOS:
60 packets transmitted, 60 received, 0% packet loss
rtt min/avg/max/mdev = 2.019/46.371/205.502/65.127 ms

Google DNS 8.8.8.8:
100 packets transmitted, 100 received, 0% packet loss
rtt min/avg/max/mdev = 769.181/2791.151/5439.359/1323.148 ms

Cloudflare DNS 1.1.1.1:
100 packets transmitted, 100 received, 0% packet loss
rtt min/avg/max/mdev = 77.020/2588.405/6091.649/1660.783 ms

nos.pt:
100 packets transmitted, 100 received, 0% packet loss
rtt min/avg/max/mdev = 508.203/2981.134/6507.877/1595.429 ms
```

Estes resultados mostram latência média entre cerca de **2,5 e 3 segundos**, com picos superiores a **6 segundos**, apesar de não existir perda de pacotes.

Solicito, por isso, a verificação técnica da rede móvel NOS na minha zona, incluindo análise de latência, jitter, routing, congestionamento ou eventual degradação do serviço.

Peço também que me indiquem o número da ocorrência/processo associado a este pedido.

Com os melhores cumprimentos,
**André Pinto**
N.º de cliente/contrato: __________
Contacto: __________
Localização dos testes: __________
Data: ___ / ___ / ______
