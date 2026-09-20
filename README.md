# Loja
Projeto feito com IA (Claude) pra teste de uma possibilidade pessoal

================================================================================
CASA DE SENSAÇÕES - LOJA ONLINE PRÓPRIA
Documentação do projeto
Gerada em 20/09/2026
================================================================================

Este arquivo explica como a loja funciona por dentro: o que cada peça faz, como
elas conversam, onde ficam os dados e como mudar as coisas. Para colocar a loja
no ar, use o arquivo GUIA-DE-APLICACAO.md (passo a passo). Este aqui é para
consulta.


--------------------------------------------------------------------------------
1. VISÃO GERAL
--------------------------------------------------------------------------------

É uma loja para vender no boca a boca, fora do Mercado Livre. O cliente escolhe
os produtos no site, informa os dados de entrega, paga por Pix e anexa o
comprovante. Você recebe tudo numa planilha, confere o pagamento, atualiza o
status e o cliente é avisado por e-mail e WhatsApp. Ele também pode acompanhar o
pedido no próprio site.

Não existe servidor para manter nem mensalidade obrigatória:

  - O SITE é um único arquivo (index.html) mais uma pasta de fotos, hospedado
    de graça (Netlify, Cloudflare Pages ou GitHub Pages).
  - O "CÉREBRO" é um script do Google (Apps Script) ligado a uma planilha do
    Google Sheets. O script recebe os pedidos, controla o estoque, envia
    e-mails e responde às consultas do site.
  - A PLANILHA é onde você trabalha no dia a dia: produtos, estoque, pedidos.

O que o projeto faz:
  - Vitrine com busca, categorias, página de detalhes e galeria de fotos
  - Sacola, formulário de entrega, Pix "copia e cola" com QR Code, comprovante
  - Estoque que desce sozinho a cada pedido e volta ao cancelar
  - E-mail automático ao cliente a cada mudança de status
  - Link de WhatsApp com mensagem pronta (você clica e envia)
  - Rastreio do pedido por código + telefone
  - Avisos para você (pedido sem estoque, valor diferente do esperado)


--------------------------------------------------------------------------------
2. ARQUIVOS DO PROJETO
--------------------------------------------------------------------------------

  index.html            O site inteiro (HTML, estilo e programação num só
                        arquivo). Vai para a hospedagem junto com a pasta img.
  apps-script.gs        O script. Não vai para a hospedagem: é colado dentro do
                        Apps Script da sua planilha (Extensões > Apps Script).
  reduzir-fotos.py      Ferramenta opcional (Python) que deixa as fotos leves.
  GUIA-DE-APLICACAO.md  Passo a passo para colocar tudo no ar.
  DOCUMENTACAO.txt      Este arquivo.
  img/                  Pasta com as fotos dos produtos (você cria; fica ao lado
                        do index.html).

Fora dos arquivos, o projeto usa:
  - Uma planilha do Google, com as abas "Pedidos" e "Produtos"
  - Uma pasta no seu Google Drive chamada "Comprovantes - Casa de Sensações",
    criada sozinha, com os comprovantes de pagamento
  - Sua conta Gmail, para enviar os e-mails


--------------------------------------------------------------------------------
3. COMO AS PEÇAS SE CONECTAM
--------------------------------------------------------------------------------

    CLIENTE (navegador)                         VOCÊ (planilha)
    ┌──────────────────┐                        ┌────────────────────────┐
    │  index.html      │                        │  Google Sheets         │
    │  (vitrine, sacola│                        │   aba Produtos         │
    │   Pix, rastreio) │                        │   aba Pedidos          │
    └────────┬─────────┘                        └───────────┬────────────┘
             │  pede a lista de produtos (GET)              │
             │  envia pedido (POST)                         │ lê e grava
             │  consulta rastreio (POST)                    │
             ▼                                              │
    ┌──────────────────────────────────────────────────────┴───────────┐
    │  Google Apps Script (apps-script.gs)                             │
    │   - entrega a lista de produtos                                  │
    │   - valida o pedido, confere preço e estoque, grava na planilha  │
    │   - guarda o comprovante no Drive                                │
    │   - envia e-mails pelo Gmail                                     │
    │   - "escuta" suas edições na planilha (gatilho)                  │
    └──────────────────────────────────────────────────────────────────┘

O site só conversa com o script, por um único endereço (o "link /exec"). Esse
endereço fica em CONFIG.appsScriptUrl, no index.html.

Você conversa com o script indiretamente, editando a planilha.


--------------------------------------------------------------------------------
4. A PLANILHA
--------------------------------------------------------------------------------

4.1 ABA "PRODUTOS"  (uma linha por produto)
--------------------------------------------

O script lê esta aba pelo NOME das colunas (a ordem não importa, mas não renomeie
os títulos abaixo). Colunas extras suas (custo, fornecedor...) são ignoradas e
nunca vão para o site.

  Código         Único e curto (P001, P002...). Nunca mude depois de criado:
                 os pedidos guardam esse código. OBRIGATÓRIA
  Ativo          Sim = aparece no site. Não = escondido. OBRIGATÓRIA
  Título         Nome do produto. OBRIGATÓRIA
  Marca          Opcional
  Categoria      Vira os filtros do site. Escreva sempre igual. Se ficar em
                 branco, o site usa "Outros"
  Preço          Em reais. OBRIGATÓRIA (produto com preço 0 não aparece)
  Preço antigo   Opcional. Se for maior que o preço, aparece riscado e o site
                 calcula a % de desconto
  Estoque        Quantidade disponível. OBRIGATÓRIA. Desce sozinho a cada
                 pedido. Em 0, o produto aparece como "Esgotado"
  Descrição      Texto livre. Linha em branco separa parágrafos
  Detalhes       Um por linha (ou separados por ";"). Se tiver "Chave: valor",
                 a chave aparece em negrito. Ex.: "Volume: 100 ml"
  Fotos          Nomes dos arquivos, separados por vírgula. A primeira é a
                 principal. Ex.: "P001-1.jpg, P001-2.jpg"
  Destaque       Sim = aparece antes dos outros na vitrine

Preços aceitam 49,25 ou 49.25. Se alguma coluna obrigatória faltar, o endereço
/exec?action=produtos devolve uma mensagem de erro dizendo qual.

4.2 ABA "PEDIDOS"  (uma linha por pedido, criada pelo script)
-------------------------------------------------------------

IMPORTANTE: o script usa a POSIÇÃO destas colunas. Não mova, não apague e não
troque a ordem. Você pode mudar o texto do título, a largura e a cor.

  Col  Título                            Quem preenche / o que é
  ---  --------------------------------  ---------------------------------------
  A    Pedido                            Código do pedido (ex.: CS260919K3F9)
  B    Recebido em                       Data e hora (automático)
  C    Nome                              Cliente
  D    Telefone                          Cliente
  E    Endereço                          Rua e número
  F    Bairro                            Cliente
  G    Complemento / referência          Cliente
  H    Itens                             Ex.: "2x Kit Pitaya (R$ 49,25)"
  I    Subtotal                          Soma dos produtos
  J    Entrega                           VALOR da taxa de entrega (R$)
  K    Total                             Subtotal + taxa
  L    Comprovante                       Link do arquivo no Drive
  M    Status do pagamento               VOCÊ escolhe: Aguardando conferência,
                                         Pago, Recusado
  N    Pago em (conferido)               VOCÊ preenche a data quando confere
  O    Entrega                           VOCÊ escolhe: Pendente, Em preparo,
                                         Saiu para entrega, Entregue, Cancelado
  P    Observações do cliente            Cliente
  Q    Avisar no WhatsApp                Link automático (fórmula)
  R    E-mail do cliente                 Cliente (opcional)
  S    Último status enviado por e-mail  Automático (evita e-mail repetido)
  T    Reserva de estoque (não editar)   Automático. Ex.: {"P001":2}
  U    Estoque devolvido                 Automático: "Sim" após cancelar
  V    Aviso do sistema                  Automático (linha fica rosa)

Atenção: existem DUAS colunas com o título "Entrega". A J é o valor da taxa; a O
é a situação da entrega. Se quiser evitar confusão, pode renomear o título da J
para "Taxa de entrega" (o script não usa o título, só a posição).

Colunas M e O têm listas para escolher (validação de dados).


--------------------------------------------------------------------------------
5. REGRAS DE NEGÓCIO
--------------------------------------------------------------------------------

5.1 STATUS ÚNICO USADO NOS AVISOS
---------------------------------

As colunas M (pagamento) e O (entrega) são combinadas num status só, para e-mail,
WhatsApp e rastreio. A prioridade é de cima para baixo (a primeira que valer):

  O = Entregue             -> "entregue"
  O = Cancelado            -> "cancelado"
  O = Saiu para entrega    -> "saiu"
  O = Em preparo           -> "preparo"
  M = Recusado             -> "recusado"
  M = Pago                 -> "pago"
  qualquer outro caso      -> "aguardando"

Ou seja: a Entrega manda mais que o Pagamento.

5.2 ESTOQUE
-----------

  Pedido novo        O estoque desce na hora (reserva). O pedido guarda o que foi
                     reservado na coluna T.
  Cancelado (O)      O estoque volta para a aba Produtos, UMA vez só (coluna U
                     passa a "Sim"). Editar de novo não devolve em dobro.
  Recusado (M)       NÃO devolve. O cliente pode mandar outro comprovante. Se
                     você desistir do pedido, marque Cancelado.
  Reativar pedido    Se você tirar um pedido de Cancelado, o estoque é reservado
                     de novo. Se não houver estoque suficiente, o pedido volta
                     sozinho para Cancelado e aparece um aviso na tela.
  Sem estoque        Se dois clientes pagam a última unidade quase juntos, o
                     segundo pedido é REGISTRADO (ele já pagou), mas entra como
                     Cancelado, a linha fica rosa e a coluna V diz "SEM ESTOQUE
                     ... Devolver o Pix de R$ X". Nenhum item é descontado (tudo
                     ou nada) e o cliente não recebe e-mail de confirmação.
  Valor diferente    Se o preço mudou entre o cliente ver e pedir, vale o preço
                     ATUAL da planilha, e a coluna V avisa "VALOR DIFERENTE ...
                     Confira o Pix".
  Reposição          Some no campo Estoque da aba Produtos.

Estoque do Mercado Livre é separado e não conversa com este.

5.3 PREÇO E VALIDAÇÃO
---------------------

O preço que vale é sempre o da aba Produtos, nunca o que o navegador enviou.
Quantidade por item: inteiro de 1 a 99. Produto inativo ou inexistente é
recusado.

O total do pedido também é recalculado pelo script (produtos + TAXA_DE_ENTREGA).

5.4 CÓDIGO DO PEDIDO
--------------------

Formato: CS + AAMMDD + 4 letras/números aleatórios. Ex.: CS260919K3F9.
É usado como número do pedido, como identificador do Pix (txid) e na consulta de
rastreio. Se o mesmo código chegar duas vezes (reenvio por falha de internet), o
script ignora a repetição.


--------------------------------------------------------------------------------
6. O SCRIPT (apps-script.gs)
--------------------------------------------------------------------------------

6.1 CONSTANTES NO TOPO (o que você pode ajustar)
------------------------------------------------

  NOTIFY_EMAIL              Seu e-mail. Recebe um aviso a cada pedido novo.
                            Vazio = sem aviso.
  ENVIAR_EMAIL_AUTOMATICO   true/false. false pausa TODOS os e-mails aos clientes.
  NOME_DA_LOJA              Nome que aparece como remetente e nos e-mails.
  URL_DA_LOJA               Endereço do site (aparece no rodapé dos e-mails).
  SHEET_NAME / SHEET_PRODUTOS   Nomes das abas ("Pedidos" / "Produtos").
  TAXA_DE_ENTREGA           Taxa em reais. 0 = grátis. O site usa este valor.
  EXIGIR_ID_DE_PRODUTO      false = ainda aceita pedidos do site antigo (sem
                            código de produto). true = todo pedido precisa
                            passar pela conferência de preço e estoque.
                            Troque para true depois que o site novo estiver no ar.
  FOLDER_NAME               Nome da pasta de comprovantes no Drive.

6.2 ENDEREÇOS QUE O SCRIPT ATENDE (todos pelo mesmo link /exec)
---------------------------------------------------------------

  GET  ?action=produtos
       Devolve os produtos ativos: id, titulo, marca, categoria, preco,
       precoAntigo, estoque, descricao, detalhes[], fotos[], destaque, mais
       config.taxaEntrega e a hora da atualização. Guardado em cache por 2
       minutos (ver 6.5). Sem parâmetro, devolve só um texto "recebimento de
       pedidos ativo" (serve para testar o link).

  POST (corpo = JSON) com dados de pedido
       Registra um pedido. Campos: code, customer{name, phone, email, street,
       district, complement, notes}, items[{id, name, qty, price}], subtotal,
       delivery, total, receipt{name, type, base64}, website (campo-armadilha
       contra robôs; deve vir vazio).
       Responde: {ok, code, semEstoque, aviso}.

  POST (corpo = JSON) com {action:"track", code, phone}
       Consulta de rastreio. Responde: {ok, codigo, status, itens[], total,
       criadoEm} ou {ok:false, error}.

6.3 O QUE ACONTECE QUANDO CHEGA UM PEDIDO (doPost)
--------------------------------------------------

  1. Descarta se o campo-armadilha veio preenchido (robô).
  2. Confere dados mínimos (código, nome, telefone, rua, itens).
  3. Trava a planilha por até 20 segundos, para dois pedidos não se atropelarem.
  4. Se o código já existe, responde ok sem gravar de novo.
  5. Planeja o pedido (planejarPedido): lê a aba Produtos, valida cada item,
     calcula o preço e o total, verifica o estoque, monta a reserva. Nada é
     alterado ainda.
  6. Salva o comprovante na pasta do Drive.
  7. Grava a linha na aba Pedidos, coloca a fórmula do WhatsApp e, se houver
     aviso, pinta a linha de rosa.
  8. Só então desconta o estoque (assim, se algo falhar antes, nada é descontado).
  9. Limpa o cache da lista de produtos.
 10. Envia o e-mail de confirmação ao cliente (se informou e-mail válido e havia
     estoque) e o aviso para você (NOTIFY_EMAIL). Falha de e-mail nunca impede o
     pedido de ser registrado.

6.4 GATILHO (aoEditarPlanilha)
------------------------------

É um gatilho "ao editar" instalado na planilha. Só dispara quando VOCÊ edita à
mão (não quando o próprio script grava).

  - Editou a aba Produtos: limpa o cache, então o site já enxerga a mudança.
  - Editou M (pagamento) ou O (entrega) na aba Pedidos:
       1. ajustarEstoquePorStatus: devolve ou reserva estoque (regras da 5.2);
       2. enviarEmailDaLinha: envia o e-mail do novo status, uma vez por status
          (a coluna S lembra o último status avisado).

O gatilho é criado pelas funções de configuração (6.6). Não há duplicação: elas
apagam o antigo antes de criar o novo.

6.5 CACHE
---------

A lista de produtos fica guardada por 120 segundos para a planilha não ser lida
a cada visita. É guardada em pedaços (o Google limita cada item a 100 KB).
Limpa sozinha quando: chega pedido, você edita a aba Produtos ou um cancelamento
muda o estoque. Se você alterar a planilha por importação ou por outro script, a
mudança pode levar até 2 minutos para aparecer.

6.6 FUNÇÕES PARA RODAR MANUALMENTE (uma vez, no editor do Apps Script)
----------------------------------------------------------------------

  configurarTudo        Faz as três abaixo. É a única que você precisa rodar.
  configurarWhatsApp    Cria a coluna Q, os novos status de entrega e preenche a
                        fórmula em todos os pedidos existentes. Rode de novo se
                        mudar o texto das mensagens de WhatsApp.
  configurarEmail       Cria as colunas R e S e liga o gatilho.
  configurarProdutos    Cria a aba Produtos (com um produto de exemplo, só se a
                        aba não existir ou estiver vazia), as colunas T, U e V da aba
                        Pedidos e liga o gatilho.

Depois de EDITAR o código do script, é preciso publicar uma NOVA VERSÃO:
Implantar > Gerenciar implantações > lápis > Versão: Nova versão > Implantar.
Sem isso, o site continua usando a versão antiga. Use sempre "Nova versão" (e
não "Nova implantação") para o link não mudar.

6.7 MAPA DAS FUNÇÕES
--------------------

  Recebimento         doGet, doPost, getSheet, getFolder, safe, json
  Produtos            lerProdutos, listarProdutos, planejarPedido,
                      cachePutBig / cacheGetBig / cacheDropBig
  Estoque             mexerNoEstoque, ajustarEstoquePorStatus
  Status              statusDoPedido
  WhatsApp            waFormula, configurarWhatsApp
  E-mail              EMAIL_TEXTOS, montarEmail, enviarEmailStatus,
                      enviarEmailDaLinha, emailValido, htmlSeguro, reais
  Rastreio            rastrear, digitosFone
  Gatilho e ajustes   aoEditarPlanilha, criarGatilho, configurarEmail,
                      configurarProdutos, configurarTudo
  Utilitários         arred, semAcento, sim, numero, partes


--------------------------------------------------------------------------------
7. O SITE (index.html)
--------------------------------------------------------------------------------

7.1 CONFIGURAÇÃO (bloco CONFIG, no início do script do arquivo)
---------------------------------------------------------------

  appsScriptUrl   Link /exec do script. Vazio = MODO DE TESTE: aparecem produtos
                  de exemplo (SAMPLE_PRODUCTS) e nenhum pedido é enviado.
  pix.key         Chave Pix que recebe (CPF/CNPJ só números, e-mail, celular
                  no formato +5511999999999 ou chave aleatória)
  pix.receiverName  Nome do recebedor: até 25 caracteres, sem acento
  pix.city        Cidade do recebedor: até 15 caracteres, sem acento
  deliveryFee     Só vale no modo de teste. Com o script ligado, a taxa vem do
                  TAXA_DE_ENTREGA do script.
  whatsapp        Opcional. Número com DDD, sem símbolos (5511999999999). Se
                  preenchido, aparece um link de contato no rodapé e na tela
                  final do pedido.
  imgFolder       Pasta das fotos (padrão "img/").

Também no início: CATEGORIES (cor de fundo por categoria, para produtos sem
foto) e PALETTE (cores automáticas para categorias novas).

As cores e fontes do visual ficam nas variáveis do bloco ":root" do estilo
(--berry, --honey, --ink, --bg...).

7.2 TELAS E FLUXO DO CLIENTE
----------------------------

  Vitrine          Título, busca, filtros de categoria, cartões de produto.
  Produto          Página de detalhes (item 7.3).
  Sacola           Painel lateral. Lista os itens, ajusta quantidade, mostra
                   subtotal, entrega e total.
  Seus dados       Nome, telefone (com DDD), e-mail (opcional), rua e número,
                   bairro, complemento, observações. Os dados ficam guardados no
                   aparelho do cliente para a próxima compra.
  Conferência      Ao avançar, o site pede a lista ATUAL de produtos ao script e
                   compara com a sacola. Se o estoque ou o preço mudou, volta à
                   sacola com um aviso, ANTES de o cliente pagar. Se a internet
                   falhar nesse momento, segue em frente (o script confere de
                   novo ao receber o pedido).
  Pagamento        Resumo, código Pix "copia e cola" com o valor, QR Code e
                   anexo do comprovante (foto ou PDF).
  Envio            Manda o pedido ao script. Se a resposta não puder ser lida,
                   reenvia uma vez (o script ignora pedido repetido).
  Tela final       Mostra o código do pedido. Se o pedido caiu no caso "sem
                   estoque", mostra uma mensagem explicando que o Pix será
                   devolvido. Tem o botão "Acompanhar este pedido".
  Acompanhar       Pede código + telefone e mostra uma linha do tempo:
                   Pedido recebido, Pagamento confirmado, Em preparo, Saiu para
                   entrega, Entregue. Mostra avisos próprios para Recusado e
                   Cancelado, mais os itens e o total.

7.3 PÁGINA DO PRODUTO E LINKS
-----------------------------

Cada produto tem um link próprio no formato  endereco-do-site/#/p/CÓDIGO
(ex.: #/p/P001). O botão "Compartilhar este produto" usa o compartilhamento do
celular ou copia o link. Abrir o link direto funciona (o site espera carregar os
produtos e mostra o certo). Código que não existe (ou produto desativado) mostra
"Produto não encontrado".

A página mostra: galeria (deslizar no celular, botões e miniaturas no
computador, contador "1 / 3"), categoria, título, marca, preço (com preço
antigo e % de desconto), estoque, botão de comprar, descrição, detalhes e até 4
produtos da mesma categoria.

Regras de estoque no texto: até 3 unidades aparece no cartão ("Restam N
unidades" / "Última unidade"); na página do produto, até 5.

7.4 O QUE FICA GUARDADO NO APARELHO DO CLIENTE (localStorage)
-------------------------------------------------------------

  cs_cart       A sacola (códigos e quantidades)
  cs_customer   Dados do formulário, para preencher de novo na próxima compra
  cs_last       Código do último pedido (preenche o campo do rastreio)
  cs_products   Cópia da última lista de produtos. Só é usada se a internet
                falhar ao carregar a loja, por até 24 horas

Nada disso sai do aparelho, a não ser o pedido enviado. Se o navegador bloquear o
armazenamento, a loja continua funcionando, só não "lembra" os dados.

7.5 PIX
-------

O site gera um Pix "copia e cola" ESTÁTICO (padrão BR Code do Banco Central) com:
chave, valor do pedido, nome, cidade e o código do pedido como identificador. O
CRC16 do código é calculado no próprio site. O QR Code vem de uma biblioteca
pública (qrcodejs, via cdnjs); se ela não carregar, o QR some e o "copia e cola"
continua funcionando.

Não há confirmação automática de pagamento: quem confirma é VOCÊ, olhando o
extrato do banco e comparando com o total do pedido.

7.6 FOTOS
---------

Ficam na pasta img, ao lado do index.html. Na planilha vai só o nome do arquivo
(com a extensão). Recomendado: nome = código do produto + número
(P001-1.jpg, P001-2.jpg), no máximo 1200 pixels, formato JPG. O reduzir-fotos.py
faz isso em lote. Se o arquivo não existir, o site mostra um bloco colorido com o
nome da marca no lugar.

Comprovantes enviados pelos clientes: fotos são reduzidas no navegador (até 1400
pixels, JPG) antes do envio; PDFs aceitam até 3 MB.

7.7 ACESSIBILIDADE E CELULAR
----------------------------

Foco de teclado visível, painéis com fechamento pelo Esc, textos para leitores de
tela nos botões de ícone e nas etapas do rastreio, respeito à preferência de
"reduzir animações". O layout foi pensado para celular primeiro.


--------------------------------------------------------------------------------
8. AVISOS AO CLIENTE
--------------------------------------------------------------------------------

8.1 E-MAIL (automático)
-----------------------

Enviado do SEU Gmail, com o nome da loja como remetente. Se o cliente responder,
a resposta chega para você. Só sai se o cliente informou e-mail válido.

  Quando:  (a) ao chegar o pedido ("Recebemos seu pedido"), exceto no caso sem
               estoque;
           (b) quando você muda o status na planilha (Pago, Em preparo, Saiu
               para entrega, Entregue, Recusado, Cancelado).
  Repetição: cada status é avisado só uma vez por pedido.
  Textos:  ficam em EMAIL_TEXTOS, no script (assunto, título e mensagem).
  Limite:  conta Gmail comum envia cerca de 100 e-mails por dia. O aviso para
           você (NOTIFY_EMAIL) também conta.
  Pausar:  ENVIAR_EMAIL_AUTOMATICO = false.

Cuidado: marcar um status por engano dispara o e-mail na hora. Os e-mails podem
cair no spam nas primeiras vezes.

8.2 WHATSAPP (semi-automático)
------------------------------

A coluna Q de cada pedido tem um link. Ao clicar, o WhatsApp abre com a conversa
do cliente e a mensagem já escrita conforme o status; você só aperta enviar. O
número é o telefone informado pelo cliente, ajustado para o formato do WhatsApp
(só dígitos, com 55 na frente). Se o cliente informou um número sem WhatsApp, o
link não vai funcionar.

Os textos ficam dentro da função waFormula. Depois de mudar um texto, rode
configurarWhatsApp para atualizar as linhas antigas (pedidos novos já usam o
texto novo).

8.3 RASTREIO
------------

O cliente digita código do pedido + telefone. O script só responde se os DOIS
baterem (o telefone é comparado só pelos dígitos, ignorando +55, parênteses e
traços). Devolve apenas: status, itens, total e data. Nunca endereço, e-mail ou
comprovante. A mensagem de erro é a mesma para "código errado" e "telefone
errado".

Proteção contra tentativa de adivinhar: depois de 40 consultas erradas em 10
minutos, o rastreio pausa por alguns minutos (o bloqueio vale para todos).


--------------------------------------------------------------------------------
9. SEGURANÇA E PRIVACIDADE
--------------------------------------------------------------------------------

Dados coletados do cliente: nome, telefone, e-mail (opcional), endereço,
observações, itens do pedido e o comprovante de pagamento. São usados só para
entregar e avisar sobre o pedido (o rodapé do site diz isso ao cliente). Estão na
sua planilha e no seu Drive.

Medidas já implementadas:
  - Preço e total vêm sempre da planilha, não do navegador.
  - Estoque conferido no servidor, dentro de uma trava.
  - Campo-armadilha contra robôs no formulário.
  - Texto digitado pelo cliente que pareça fórmula (começa com = + - @) é neutralizado
    antes de ir para a planilha.
  - Conteúdo dos e-mails é escapado (sem HTML injetado).
  - A lista pública de produtos só contém as colunas previstas. Custo,
    fornecedor e outras colunas suas nunca saem.
  - Rastreio exige código + telefone e devolve dados mínimos.

O que vale saber:
  - O link /exec é público (fica no código do site) e o script roda com o seu
    acesso. Por isso, NÃO dê acesso de edição da planilha ou do Apps Script a
    quem você não confia.
  - O script tem permissão para Planilhas, Drive, envio de e-mail e gatilhos da
    sua conta. Foi isso que você autorizou na primeira execução.
  - Não guarde pedidos e comprovantes para sempre sem necessidade. De tempos em
    tempos, apague dados antigos que você não precisa mais (LGPD: guardar só o
    necessário e pelo tempo necessário). Isto não é assessoria jurídica; se tiver
    dúvida sobre obrigações fiscais ou de dados, consulte seu contador.
  - Direito de arrependimento: o cliente pode desistir em até 7 dias após
    receber, conforme o Código de Defesa do Consumidor (o rodapé informa isso).


--------------------------------------------------------------------------------
10. ROTINA DO DIA A DIA
--------------------------------------------------------------------------------

  Chegou um pedido
    1. Abra a aba Pedidos. Confira o Pix no seu banco (valor e nome).
    2. Marque M (Status do pagamento) = Pago e preencha N com a data.
       -> o cliente recebe o e-mail "Pagamento confirmado".
    3. Clique no link da coluna Q para avisar também pelo WhatsApp.

  Andamento
    - Mude O (Entrega): Em preparo, Saiu para entrega, Entregue.
      Cada mudança envia o e-mail; o link do WhatsApp muda a mensagem sozinho.

  Pagamento não localizado
    - Marque M = Recusado. O estoque continua reservado. Se o cliente desistir,
      marque O = Cancelado (o estoque volta).

  Linha rosa
    - Leia a coluna V. "SEM ESTOQUE": devolva o Pix. "VALOR DIFERENTE": confira o
      Pix contra o total.

  Produtos
    - Novo produto: nova linha na aba Produtos (código novo e único).
    - Preço, estoque, descrição: edite a célula. Aparece no site na hora.
    - Tirar da loja: Ativo = Não.
    - Chegou mercadoria: some no Estoque.

  De vez em quando
    - Apague pedidos e comprovantes antigos que não precisa mais.
    - Confira se os produtos "Ativos" ainda existem fisicamente.


--------------------------------------------------------------------------------
11. COMO FAZER MUDANÇAS COMUNS
--------------------------------------------------------------------------------

  Trocar textos dos e-mails       Script: EMAIL_TEXTOS. Nova versão.
  Trocar textos do WhatsApp       Script: waFormula. Nova versão + rodar
                                  configurarWhatsApp.
  Mudar a taxa de entrega         Script: TAXA_DE_ENTREGA. Nova versão. (O site
                                  passa a mostrar o valor sozinho.)
  Pausar e-mails                  Script: ENVIAR_EMAIL_AUTOMATICO = false.
  Mudar chave Pix / nome / cidade Site: CONFIG.pix. Subir o site de novo.
  Mudar cores                     Site: variáveis em ":root" no estilo.
  Mudar o nome da loja            Site: <title>, cabeçalho, título da página
                                  inicial, textos do rodapé e CONFIG.pix
                                  .receiverName. Script: NOME_DA_LOJA.
  Nova categoria                  Basta digitar na coluna Categoria. A cor é
                                  automática (ou coloque em CATEGORIES).
  Mudar o texto do rodapé         Site: bloco <footer> no fim do HTML.

  Adicionar um STATUS novo (mais trabalhoso; precisa alterar em 5 lugares):
    1. Script: DELIVERY_STATUSES e a lista de validação (configurarWhatsApp);
    2. Script: statusDoPedido;
    3. Script: waFormula (mensagem de WhatsApp);
    4. Script: EMAIL_TEXTOS;
    5. Site: TRACK_STEPS (etapa na linha do tempo do rastreio).

  Regra geral:
    - Mexeu no SCRIPT  -> salvar e publicar uma NOVA VERSÃO da implantação.
    - Mexeu no SITE    -> subir de novo a pasta na hospedagem e recarregar a
                          página no navegador.
    - Mexeu na PLANILHA (produtos) -> nada a publicar.


--------------------------------------------------------------------------------
12. LIMITES CONHECIDOS
--------------------------------------------------------------------------------

  - Confirmação do Pix é manual (você confere no banco).
  - Se duas pessoas pagam a última unidade juntas, uma precisa de devolução. O
    site avisa e a planilha marca, mas o dinheiro você devolve.
  - Estoque próprio, separado do Mercado Livre.
  - Prévia de link de produto no WhatsApp mostra o nome da loja, não a foto do
    produto (as prévias não executam o programa do site).
  - O cliente vê um pedido por consulta. Histórico de tudo que já comprou exigiria
    login (não implementado de propósito).
  - Gmail comum: cerca de 100 e-mails por dia.
  - O rastreio pausa para todos depois de muitas tentativas erradas seguidas.
  - Entrega só na cidade, com taxa única. Sem cálculo de frete por bairro.
  - Depende do Google (Sheets, Apps Script, Drive, Gmail) estar no ar e das
    cotas gratuitas dele.
  - Os e-mails saem do seu Gmail e podem ir para o spam do cliente.


--------------------------------------------------------------------------------
13. SOLUÇÃO DE PROBLEMAS
--------------------------------------------------------------------------------

  Site mostra "Modo de teste"
    -> CONFIG.appsScriptUrl está vazio ou errado.

  Produtos não aparecem
    -> Abra o link /exec?action=produtos no navegador. Se mostrar erro, ele diz
       qual coluna falta. Se a lista vier vazia, veja se há produtos com
       Ativo = Sim, Título e Preço maior que zero.

  Foto não aparece
    -> Nome na planilha diferente do arquivo (maiúsculas contam; inclua .jpg), ou
       a pasta img não foi enviada junto na hospedagem.

  Pedido não entra na planilha
    -> Link do script incompleto; implantação sem acesso "Qualquer pessoa"; ou
       EXIGIR_ID_DE_PRODUTO = true com o site antigo.

  Mudei o script e nada mudou
    -> Faltou publicar uma NOVA VERSÃO da implantação.

  Mudei o site e nada mudou
    -> Faltou subir de novo na hospedagem (ou atualizar a página).

  E-mail não chega
    -> Cliente não informou e-mail; ENVIAR_EMAIL_AUTOMATICO = false; o status já
       tinha sido avisado antes (coluna S); cota diária; ou caiu no spam. Se o
       gatilho sumiu, rode configurarEmail de novo.

  Link do WhatsApp mostra erro na célula
    -> Rode configurarWhatsApp. Confira o telefone da linha (coluna D).

  O estoque não voltou ao cancelar
    -> O pedido não tinha reserva (pedido de teste antigo, pedido do site antigo
       ou sem estoque). Confira a coluna T. Ajuste na mão na aba Produtos.

  "Um dos produtos não está mais disponível" ao pedir
    -> O produto foi desativado ou o código mudou entre a escolha e o envio. Se o
       cliente já pagou, combine a devolução ou a troca.

  Cliente diz que o rastreio não acha o pedido
    -> Confira se ele digitou o mesmo telefone da compra e o código exatamente
       como veio; se houve muitas tentativas erradas, aguarde alguns minutos.


--------------------------------------------------------------------------------
14. POR QUE FOI FEITO ASSIM (decisões)
--------------------------------------------------------------------------------

  Google Sheets + Apps Script
    Gratuito, sem servidor para cuidar, e você edita produtos e pedidos numa
    planilha que já sabe usar.

  Sem cadastro/login de cliente
    Cadastro afasta compradores e traz mais responsabilidade com dados. O rastreio
    por código + telefone resolve o principal (saber como está o pedido).

  WhatsApp semi-automático
    A API oficial tem custo e aprovação de mensagens, e opções não oficiais podem
    banir o número. O link com mensagem pronta é gratuito e seguro.

  E-mail opcional
    Quem não quer informar continua comprando, e recebe aviso pelo WhatsApp.

  Estoque descontado no pedido, não na conferência do Pix
    Entre o pedido e a conferência podem passar horas; nesse tempo a última
    unidade poderia ser vendida duas vezes. Cancelado devolve; Recusado não, pois
    o cliente costuma mandar outro comprovante.

  Fotos na pasta do site, não links do Drive
    Links do Drive são lentos e falham com muito acesso. Com fotos no próprio
    site, elas carregam rápido e não somem.

  Pix estático com conferência manual
    Simples e sem custo por transação. A evolução natural, se o volume crescer, é
    a confirmação automática pela API de pagamentos (ex.: Mercado Pago).

  EXIGIR_ID_DE_PRODUTO
    Permite trocar primeiro o script e depois o site sem derrubar a loja. Quando o
    site novo está no ar, liga-se para valer a conferência em todo pedido.


--------------------------------------------------------------------------------
15. IDEIAS PARA O FUTURO
--------------------------------------------------------------------------------

  - Pix com confirmação automática (API do Mercado Pago), eliminando a conferência
    manual e o risco de comprovante falso
  - Reserva de estoque com prazo enquanto o cliente paga
  - Login opcional com histórico de compras
  - Frete por bairro ou entrega em cidades vizinhas
  - Cupons de desconto e frete grátis acima de um valor
  - Avaliações de clientes nos produtos
  - Painel com resumo de vendas do mês
  - Imagem de prévia própria ao compartilhar link (exigiria páginas individuais
    por produto)


--------------------------------------------------------------------------------
16. GLOSSÁRIO
--------------------------------------------------------------------------------

  Apps Script      Ferramenta do Google que roda programas ligados a planilhas.
  Implantação      A "publicação" do script, que gera o link /exec.
  Nova versão      Atualização da implantação depois de editar o código.
  Gatilho          Programa que roda sozinho quando algo acontece (aqui: quando
                   você edita a planilha).
  Link /exec       Endereço público do script. O site fala com ele.
  Pix copia e cola Texto que o cliente cola no app do banco para pagar.
  Cache            Cópia temporária guardada para não ler a planilha toda hora.
  Reserva          Estoque já descontado por um pedido que ainda não foi entregue.
  Hospedagem       Serviço que deixa o site acessível na internet.
  LGPD             Lei Geral de Proteção de Dados (regras sobre dados pessoais).

================================================================================
Fim da documentação
================================================================================
