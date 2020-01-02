instale Docker e Docker-compose

curl -X POST -H 'Content-Type: application/json' http://localhost:5005/webhooks/rest/webhook -d '{"sender":"Eu","message":"olá"}'

Usando a GPU NVIDIA nos contêineres do Docker

Mergulhar no aprendizado de máquina requer algum poder computacional, trazido principalmente pelas GPUs. Mas reluto em instalar novas pilhas de software no meu laptop - prefiro instalá-las nos contêineres do Docker, para evitar poluir outros programas e poder compartilhar os resultados com meus colegas de trabalho. Isso significa que eu tenho que configurar o Docker para usar minha GPU. Esta é a história de como consegui fazê-lo, em cerca de meio dia.

Estou acostumado a usar o Docker para todos os meus projetos na marmelab. Permite configurar facilmente até as infra-estruturas mais complexas, sem poluir o sistema local. No entanto, como o processamento de imagem geralmente requer uma GPU para obter melhores desempenhos, a primeira pergunta é: o Docker pode lidar com GPUs?

Procurar uma resposta para esta pergunta me leva ao repositório nvidia-docker, descrito de maneira concisa e eficaz como:

"Crie e execute contêineres Docker utilizando as GPUs NVIDIA"

Felizmente, tenho uma placa gráfica NVIDIA no meu laptop. Os engenheiros da NVIDIA encontraram uma maneira de compartilhar os drivers da GPU do host para os contêineres, sem que eles fossem instalados em cada contêiner individualmente.

As GPUs no contêiner seriam as contêineres host. Parece promissor. Vamos tentar!

Instalando o CUDA no host

CUDA é uma plataforma de computação paralela que permite usar GPU para processamento de uso geral. É altamente recomendável ao lidar com o aprendizado de máquina, uma tarefa importante que consome recursos.

O nosso cartão gráfico suporta CUDA?

O primeiro passo é identificar com precisão o modelo do meu cartão gráfico. Isso é feito facilmente no Linux usando o utilitário lspci:

`lspci | grep VGA`

Então, eu tenho uma GeForce 840M. Verificando no site comercial da NVIDIA, este cartão tem suporte para CUDA. Ótimo até agora!

Instalando CUDA para placas gráficas NVIDIA

Vamos seguir o guia de instalação da placa gráfica NVIDIA para instalar a versão mais recente do CUDA. Estou usando o Linux Mint 18.2, baseado no Ubuntu Xenial (16.04).

Esta última etapa causou um problema devido ao protocolo https:

Corrigir é tão fácil quanto instalar o pacote ausente:

Agora eu posso instalar o pacote cuda da maneira padrão:

Depois de baixar e instalar quase 3 GB de dados, ainda preciso executar algumas tarefas pós-instalação.

Atualizando variáveis ​​de ambiente

Primeiro, preciso atualizar algumas variáveis ​​de ambiente. Eu edito meu arquivo ~ / .bashrc e adiciono as seguintes linhas:

 recarregue as variáveis ​​de ambiente usando:

Daemon de persistência NVIDIA

Então, preciso iniciar o NVIDIA Persistence Daemon como o primeiro software da NVIDIA durante o processo de inicialização. O daemon de persistência visa manter a GPU inicializada mesmo quando nenhum cliente está conectado a ela e manter seu estado nas tarefas CUDA. Nenhuma ideia precisa do motivo, mas é necessária.

Crio um novo /usr/lib/systemd/system/nvidia-persistenced.service com o seguinte conteúdo:

Então, habilito este serviço recém-criado:

Desativar alguma regra UDEV

Uma regra do udev (uma interface entre dispositivos físicos e o sistema) impede que o driver NVIDIA funcione corretamente. Portanto, preciso editar o arquivo /lib/udev/rules.d/40-vm-hotadd.rules e comentar a regra do subsistema de memória:

Após a reinicialização da máquina, é hora de testar minha instalação do CUDA, compilando alguns dos exemplos fornecidos: