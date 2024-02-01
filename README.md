# Sistemas Ciberfísicos
Neste repositório visa-se realizar uma análise teórica de possíveis ataques ou problemas de segurança atrelados à sistemas ciberfísicos para meios de aprimoramento de segurança. 

Tomamos como referência um dos casos apresentados no livro "The Hardware Hacking Handbook: Breaking Embedded Security with Hardware Attacks", mais especificamente a utilização de fault injection attacks em consoles de videogame.

## Fault Injection

A ideia usada neste caso é causar pequenos problemas durante a execução funções do dispositivo em questão, possibilitando atacar diferentes mecanismos de segurança presentes no mesmo. Neste caso, utiliza-se algum hardware externo ao dispositivo para tentar chegarmos ao resultado esperado.

## Fault Injection PS3 Hypervisor

Esse ataque por injeção de falhas idealizado por George Hotz se baseia na feature presente originalmente no console de possibilitar rodar o OS Linux se desejado. 

Nesta situação, após bootar o Linux é realizado um glitching attack com o objetivo de comprometer a Tabela de Página com Hash para conseguir acesso ao segmento responsável pelo mapeamento da memória, realizando uma desincronização entre a Tabela de Página e a RAM, podendo controlar qualquer região da memória. Neste caso, podemos afirmar que para êxito do processo é necessário uma ação conjunta entre software e hardware para execussão do exploit.
### Análise Detalhada
Primeiro aloca-se um buffer, então requisita-se a criação de vários mapeamentos da Tabela apontando para o buffer criado. Após isto, desalocamos o buffer após liberar todos os mapeamentos para ele, a ideia é que o hipervisor destrua cada um dos mapeamentos, porém se realizarmos o glitching attack neste momento, afeta-se o hipervisor de modo que o mesmo pense que houve o desalocamento da memória enquanto o mapeamento se mantém intacto.

Neste caso, o hipervisor ainda contém os mapeamentos para o buffer desalocado, podendo agora modificar o seu conteúdo. Neste ponto o exploit cria um segmento virtual e verifica se ele se localiza em alguma parte da memória referente ao endereço do buffer liberado, criando segmentos virtual até que algum esteja, possibilitando o usuário escrever diretamente para esta Tabela de Páginas, neste ponto o ataque instala duas syscalls que lhe fornecem acesso direto de leitura e escrita em qualquer endereço de memória

## PS3 Jailbreak

O ataque acima abriu possibilidades para outras formas de explorar o console, dando origem ao uso de jailbreaks, que permitem o usuário a jogar jogos piratas. Como analisado por E. Debusschere e Mike McCambridge no artigo Modern Game Console Exploitation. O ataque se baseia em uma vulnerabilidade na verificação de dispositivos USB do console.

A ideia é plugar um dispositivo USB no console e se aproveitar do tempo durante o boot que o console leva para inicializar os dispositivos USB, o dispositivo responsável pelo jailbreak então simula um hub USB com 6 portas, de modo a plugar e desplugar dispositivos nele até que ocorra um corrompimento do heap de modo a ser explorado.

Primeiro, simula-se que as 3 primeiras portas do hub estão ocupadas, logo após manda um sinal informando que o dispositivo na porta 2 foi removido. Após isto um dispositivo é "plugado" na porta 4 com 3 descriptors possibilitando se aproveitar de vulnerabilidades no heap.

Após mais altrerações nos descritores da porta 4 e a análise sintática do payload contido no descritor da porta 2 que já foi desplugada, o PS3 acaba por sobrescrever a tag de limite do malloc no começo do descrito da porta 3. Neste caso o ataque é iniciado quando a tag de limite do malloc é apontada para uma função chamada quando um dispositivo USB é liberado.

Por fim, o dispositivo envia um sinal de que um dispositivo oficial de PS3 foi conectado à porta 5, uma key é enviada para o dispositivo que responde com dados estáticos, estabelecendo código desconhecido na memória, o hub então desconecta o suposto dispositivo na porta 3, chamando a função sobrescrita e completa o ataque.
