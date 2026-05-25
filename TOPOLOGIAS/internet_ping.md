

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

