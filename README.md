# temperatura-cpu-zabbix-windows
![Title](images/zabbix.jfif)

*Integrando monitoramento de temperatura da CPU com o Zabbix no Windows*

Ao criar um lab de monitoramento, uns dos principais índices a ser monitorado é a temperatura do seu servidor, como já sabemos, a temperatura influencia diretamente no desempenho da máquina, quanto maior a temperatura, a CPU diminui o Clock perdendo desempenho com objetivo de diminuir a temperatura, em casos extremos quando a temperatura bate o limite de segurança, a máquina é desligada para proteger o hardware.

O Zabbix Agent por si só, não suporta nativamente o monitoramento da temperatura da CPU, iremos integrar softwares de terceiros com o Zabbix.
Vamos parti do princípio que o seu server Zabbix está todo já configurado e o Windows está com o Agent Zabbix instalado e coletando os dados.

**1º Passo** - Baixe as ferramentas Real Temp 3.70 e *Tail.exe* e *Gawk.exe*.

**2º Passo** - Vamos criar uma pasta chamada RealTemp_370 dentro do Disco Local C e descompacte o Real Temp para dentro dela.

**3º Passo** - Abra o *RealTemp.exe*, logo em seguida click em *Settings*.

![Title](images/1.jfif)

Note que o software já está monitorando a temperatura. No meu caso, ele está monitorando a temperatura dos 4 core da CPU.

**4º Passo** - Marque as opções "*Start Minimized*", "*Log File*" e "*Minimize on Close*" e de OK.

![Title](images/2.jfif)

**5º Passo** - O Real Temp irá coletar os dados da temperatura a cada 5 segundos e armazenar em um arquivo de Log, o Zabbix vai pegar o último valor coletado desse log e nos informar.

Vamos abrir o arquivo de log *RealTempLog.txt*

![Title](images/3.jfif)

Dentro do arquivo, vamos ter várias colunas como mostra a imagem, como minha CPU possuí 4 core, o programa vai listar a coluna CPU_0 até a CPU_3.

**DATE**: *Mostra a data coletada*
**TIME**: *Mostra a hora coletada*
**MHz**: *Mostra o clock*
**CPU_**: *O Núcleo da CPU*
**Load%**: *Mostra a % de utilização da CPU, como o Zabbix já possui nativamente o a função system.cpu.util, não iremos coletar a utilização da CPU.*

Iremos utilizar o campo CPU_0 para ter como base a temperatura do processador.

**6º Passo** - Vamos utilizar agora o Tail.exe e Gawk.exe, ao efetuar o download, abra o arquivo *UnxUpdates.zip* e descompacte o *Gawk e Tail* para dentro da pasta system32 *C:\Windows\System32*).
![Title](images/4.jfif)

![Title](images/5.jfif)

**7º Passo** - Vamos verificar a coleta. Abra o cmd.exe como ADM e execute o comando:

***tail -1 C:\RealTemp_370\Realtemplog.txt | gawk "{print $4}"***
![Title](images/6.jfif)

O Valor 31 é a temperatura atual.

**Explicação**: como jogamos o *Tail* e *Gawk* para dentro de system32, assim podemos chama-los através do cmd sem precisar de URL. Logo em seguida introduzimos a URL do log, depois introduzimos o comando para chamar o gwank com o parâmetro "*{print $4}*" para exibir na tela o valor da coluna 4 *($4)* que é nosso *CPU_0*.

Feito isso, o programa está configurado corretamente.

**8º Passo** - Na pasta de onde o Zabbix Agent foi instalado ( *C:\Program Files\Zabbix Agent* ) abra o arquivo zabbix_agentd.conf e introduza na última linha: 

***UserParameter=temp.tempcore0,tail -1 C:\RealTemp_370\Realtemplog.txt | gawk "{print $4}"***
![Title](images/7.jfif)

Salve e feche.

**9º Passo** - Em *Services.msc*, reinicie o serviço do *Zabbix Agent*.

**10º Passo** - Entre no seu Zabbix. 
![Title](images/8.jfif)

Acesse *Configurações > Templates.*

Crie um Novo *Template.*

Coloque o nome Template Temperatura Windows
![Title](images/9.jfif)

![Title](images/10.jfif)

Em Aplicações, crie uma nova aplicação e coloque o nome Template Temperatura Windows e crie.

Agora vá até ITEM e coloque as seguintes configurações conforme á imagem:

*Nome: Temperatura Core 0*
*Tipo: Agente Zabbix*
*Chave: temp.tempcore0*
*Unidade: C*
*Intervalo de atualização: 10s (você pode alterar)*
*Aplicações: Escolha Template Temperatura Windows*

![Title](images/11.jfif)

*Salve o ITEM.*

**11º Passo** - Vamos adicionar o Template ao nosso Host, Em Configurações, escolha Host.

![Title](images/12.jfif)

Dentro do Host, click em Templates e Vincule um novo modelo de Template.

![Title](images/13.jfif)

Adicione o nosso Template de temperatura e Click em Atualizar o Template do Host.
![Title](images/14.jfif)

Pronto, Monitoramento de temperatura criado =)
Vamos ver os Dados Coletados!

Em Monitoramento, click em Hosts e escolha seu Host.

![Title](images/15.jfif)

![Title](images/16.jfif)

No seu Host, escolha Dados Recentes.

Navegue até encontrar o Template Temperatura Windows, depois click em Gráfico
![Title](images/17.jfif)

![Title](images/18.jfif)

Pronto, nosso Zabbix já está coletando a temperatura do nosso Servidor. =)

![Title](images/19.jfif)
