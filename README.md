## LIVE PSN CANDELARIA COM NGINX (PARA YOUTUBE E FACEBOOK)

_Pré-Requisitos: OBS Studio instalado e [baixar os arquivos do NGINX](https://drive.google.com/open?id=1hr_uehZfe8cHS6sZTQ7JbLKHqSm_LPPw)_

#### 1. PROCEDIMENTOS PARA INICIAR TRANSMISSÃO

> Caso seja o primeiro uso no computador, fazer os **procedimentos de instalação** descritos no próximo capítulo

1. Abrir o **OBS Studio**, configurar e testar as cameras e som
2. Preparar a transmissão ao vivo nas plataformas (Youtube e Facebook), fazendo o agendamento
3. Pegar as chaves de transmissão 
4. Editar o arquivo de configuração do **nginx.conf** com os parâmetros da nova transmissão e **salvar o arquivo**.
5. Iniciar o stunnel, **stunnel GUI Start**
6. Iniciar o nginx, **nginx - Start**
7. Iniciar a transmisão no **OBS Studio**
8. Acompanhar recepção do streaming no youtube e no facebook
6. Iniciar a transmissão nas plataformas

#### 2. PROCEDIMENTOS DE INSTALAÇÃO

1. Instalar o stunnel, ele é um proxy que permite que a funcionalidade de criptografia do facebook seja suportada pelo OBS. Ou seja, transmitimos em rtmp e o facebook recebe em rtmps.
	- https://www.stunnel.org/downloads.html
	- Abrir o arquivo de configuração em _C:\Program Files (x86)\stunnel\config\stunnel.conf_
	- Copiar o texto abaixo no final do arquivo "stunnel.conf" e salvar.

		[fb-live]
		client = yes
		client = yes
		accept = 127.0.0.1:1935
		connect = rtmp-pc.facebook.com:443
		verifyChain = no


    > **Nota**: Vou fazer testes mudando o valor de connect para "live-api-s.facebook.com:443", mas na live que funcionou deu certo do jeito que está acima.

2. Iniciar o stunnel (no menu iniciar são criados alguns atalhos para "stunnel GUI start" e "stop"), após iniciar ele vai lá pra perto do relógio)

3. Copiar pasta "nginx" para C:\

4. Abrir o arquivo de configuração do nginx em _C:\nginx\conf\nginx.conf_ e procurar pelo objeto **application live**

        application live {
            live on;
            push rtmp://link-de-transmissão-youtube/chave-tranmissão-youtube;
            push rtmp://127.0.0.1:1935/rtmp/chave-de-transmissão-facebook;
        }

	- Editar os "push", cada linha de push é uma transmissão diferente
		- Para youtube basta colocar o endereço rtmp, seguido da chavede transmissão
		- Para facebook colar a chave para depois de rtmp/
		
    > **Nota 1**: No caso do facebook, o nginx vai mandar o fluxo de dados sem criptografia e o stunnel vai fazer isso pra nós e enviar para o facebook

    > **Nota 2**:  Caso não queira transmitir algum fluxo, é só comentar a linha com um # e não precisa mudar nada no OBS.

        EXEMPLO DE TRANMISSÃO SOMENTE PELO YOUTUBE COM NGINX: 
        application live {
            live on;
            push rtmp://link-de-transmissão-youtube/chave-tranmissão-youtube;
          # push rtmp://127.0.0.1:1935/rtmp/chave-de-transmissão-facebook;
        }
	
5. Salvar o arquivo de configuração e Iniciar o servidor nginx (c:\nginx\nginx.exe)
	> **Nota**: Se alguma modificação foi feita no arquivo de configuração, reiniciar o nginx

6. Copie a pasta "Controle Live" para a área de trabalho. São apenas atalhos para coisas que são importantes

7. Saiba o nome do seu computador, **digite no cmd**

	    echo %computername%

7. No OBS, em "Transmissão" colocar os seguintes parâmetros:

        Serviço: Personalizado
        Servidor: rtmp://nome-do-computador/live
        Chave de transmissão: <em branco> 
        Utilizar autenticação: <desmarcado>
	
    **Nota**: substitua nome-do-computador pelo nome do seu computador. Fiz uns testes com localhost e 127.0.0.1 no meu computador e deu umas falhas. Colocar o IP do wifi aqui não é uma boa prática porque senão vai ter que mudar o ip sempre. Caso ocorra algum problema utilizando o nome do computador, pode ser necessário criar uma interface de loopback com um endereço fixo.

9. Configuração de transmissão no OBS

        Taxa de bits de vídeo: 1800 kbps
        Intervalo de keyframe: 2
        Taxa de áudio: 128 kbps

    Um único fluxo vai ser gerado pelo OBS para o nginx, que vai duplicá-lo e os distribuir para o youtube e facebook ao mesmo tempo.

    > Atentar que para o OBS estará existindo uma única transmissão, o que auxilia em baixar a carga de processamento do processador, mas ainda estará havendo o upload de dois fluxos de vídeo para a Internet.

        Upload resultante com taxa de bits de vídeo em 1800 kbps:
            Fluxo youtube : 1800 kbps + 128 kbps = 1928
            Fluxo facebook: 1800 kbps + 128 kbps = 1928
            > Upload total: 3856 kbps (3,77 Mbps)

        Upload resultante com taxa de bits de vídeo em 2000 kbps:
            Fluxo youtube : 1800 kbps + 128 kbps = 2128
            Fluxo facebook: 1800 kbps + 128 kbps = 2128
            > Upload total: 4256 kbps (4,15 Mbps)            

        Upload resultante com taxa de bits de vídeo em 2500 kbps:
            Fluxo youtube : 2500 kbps + 128 kbps = 2628
            Fluxo facebook: 2500 kbps + 128 kbps = 2628
            > Upload total: 5256 kbps (5,14 Mbps)

_As intruções de configuração deste pequeno manual foram baseadas nos testes que fiz, novas informações são sempre bem vindas_ 😀
