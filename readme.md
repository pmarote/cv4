# Cv4
Conversor 4, reescrevendo o Conversor 3 do zero, incluindo tudo do projeto pn-PublicoNet

## Todo

- Backend: partindo do exemplo de https://github.com/gothinkster/slim-php-realworld-example-app
- Frontend: partindo do exemplo de https://github.com/rtfeldman/elm-spa-example

- Versão 4.0.x
	- trabalhar apenas na parte de pn-PublicoNet
	- Fechar apenas o backend
	- Usar apenas diários oficias tj sp 1a instância capital e 2a instância
	- Funciona em conjunto com cv3, como um módulo adicional, menu cv4

- Versão 4.1.x
	- Usar consultas web como tit público

- Versão 4.2.x
	- Frontend - Ver no final deste md

- Versão 4.3.x 
	- Começar a trazer o que está no Conversor 3 para cá


### Como funciona o threadConv ?

- Vai processando tudo o que estiver em orig (originais), mas apenas um processamento por vez. Ou seja, faz algo, encerra e volta novamente ao início. Enquanto o checkbox superior “repete as conversões enquanto houver processamento a ser feito”, ele vai fazendo. Se vc pretende desligar o computador e continuar depois, uncheck o checkbox que ele para ao encerrar o próximo.
- em orig (originais), na pasta raiz ou em uma pasta em sua escolha, devem ser jogados os diários oficiais ou qualquer outra fonte de dados, como autos de infração da Sefaz. Para ajudar a organização, recomenda-se colocar o número do dia da disponibilização no nome do arquivo, caso aplicável, e o mês e ano no nome da pasta. Nada disso é obrigatório e será utilizado mais a frente para organizar os chamados universos;
		.pdf 	- Normalmente é a regra
		.html	## todo
		.txt 	## todo
- após acabar tudo, o que não for mais usado fica em proc (processados), nas mesmas subspastas que estavam em orig (serão criadas automaticamente, caso necessário)
	- ou seja, os arquivos .pdf e .json
- na pasta res (resources ou resultados, o sentido é o mesmo), vai ficar o que vai ser usado
	- O nome do arquivo db3 será o Hash MD5 do arquivo original – está sendo feito isso para ganhar tempo e espaço, caso alguém mande o mesmo arquivo para ser processado; (##VER O QUE FAZER CASO JÁ EXISTA##)
	- Serão gerados DOIS arquivos db3:
		a) hash.db3, contendo as seguintes tabelas:
			`orig` - Tudo sobre o arquivo fonte e a pasta original 
			`fonts` - Definição das fontes originais
			`background` - Definição de qual background utilizar para cada página
			`lines_do` - conteúdo txt do pdf original e sua localização, para construir o html
			`arq_jpegs` - o conteúdo exato das diferentes imagens originais, em formato jpeg
		b) hash.idx.db3, contendo os dados necessários para o Full Text Search
- Organização da pasta res:
	- Com base nas primeiras linhas do arquivo, do nome original, da pasta original ou conforme outras regras a serem verificadas, um super indexador ira classificar os arquivos .db3 
	- Identificado o arquivo, ele joga em um db3 de índice geral e move os arquivos para dentro de pastas que serão chamados de “universos”
	- Ou seja, o que está na pasta raiz de res, não foi classificado em universo; o que está dentro de uma pasta em res, já foi classificado em um universo; caso haja necessidade de reclassificação, mova os arquivos para a rais de res e rode o super indexador;
	- Ok então. A partir de agora chamaremos de universo o nome de cada uma dessas pastas. Sugestão de convenção de nomes: 
		tjsp	- Diários oficiais de primeira e segunda instância do TJ-SP
		stj	- Diários oficiais do STJ
		stf	- Diários oficiais do STF
		trt2	- Diários oficiais de primeira e segunda instância do TRT-2
		tst	- Diários oficiais do TST
		SfSP_AIIM	- Decisões de AIIMs da Secretaria da Fazenda de SP
		SfSP_CT	- Consultoria Tributária da Secretaria da Fazenda de SP
	- dentro dessas pastas onde cada uma é um “universo”, podem ser criados subuniversos, como ano e mês, preferencialmente no formato AAMM 

- Tudo é automático e tem correção automática de erro, caso tenha dado problemas anteriormente. A correção é meio burralda, mas é assim: todo o processamento é feito dentro de _sistema\tmp e _sistema\db3 . Cada vez que é iniciada uma conversão, primeiro é chamado Reinicializa.php que vai providenciar que _sistema\tmp e _sistema\db3 sejam apagados. Como assim?
	- Ao clicar em Inicia Conversão é chamado function clickok(), que simplesmente chama function reinicializa()
	- function reinicializa() chama o thread php-win Reinicializa.php e encerra o aplicativo principal pnw.php
- Reinicializa.php basicamente fica tentando dar recursiveDelete('tmp') e  recursiveDelete('db3') e, se não conseguir sucesso, em algumas tentativas, chama , _KillProcesses.vbs que, basicamente é taskkill /f /im de php-win.exe, php.exe e sqliteman.exe, chamando em seguida novamente Reinicializa.php
- Encerrando Reinicializa.php , vai agora a _Reinicializa.bat (ou .vbs), que simplesmente chama pnw.php console reinicializa ou seja, volta ao início mas no modo reinicializa
- Reiniciando pnw.php  no modo reinicializa,ou seja, após clicar em Inicia Conversão, após reinicializar, ele printa a tela mas não para, vai direto a function principal(), que basicamente chama o thread php-win ThreadConv.php  e fica aguardando encerrar esse thread, simplesmente verificando se o thread gerou logs/ThreadConvPronto.log ou logs/ThreadConvMaisTarefas.log
- Ou seja, php-win ThreadConv.php, ao acabar, pode dois sinais: 
	a) logs/ThreadConvPronto.log vai fazer com que sossegue
b) logs/ThreadConvMaisTarefas.log vai fazer com que reinicialize, ou seja, é como se tivesse encerrado e logo em seguida fosse apertado novamente  Inicia Conversão. Isso, logicamente, conforme já explicado, apenas se estiver chegado o checkbox superior “repete as conversões enquanto houver processamento a ser feito”.

O que faz ThreadConv.php, em suma ?

Em suma
- gera os json possíveis de todos os pdfs presentes
	- gera os html de todos os pdfs presentes
	- gera todos os .db3 

- Como?
- Exige-se que dentro de orig haja pastas, no formato AAMM . Exemplo: 1805 . O conteúdo em proc e res vai refletir essas mesmas pastas. O que estiver fora dessas pastas é ignorado.
Relembrando:
- após acabar tudo, o que não for mais usado fica em proc (processados)
	- ou seja, os arquivos .pdf e .json
- na pasta res (resources ou resultados, o sentido é o mesmo), vai ficar o que vai ser usado
	- ou seja, os arquivos origem.db3 (conteúdo txt do pdf original para contruir o html), origem.jpg.db3 (imagens do pdf original para contruir o html) e origem.idx.db3 (índice Full Text Search)
As tarefas são:
	- melhora o nome do arquivo, deixando mais amigável para urls. Tira espaços e acentuações, deixando também independente de UTF8 ou ISO-8859-1;
- gera o .json em proc;
	- gera a pasta html em proc, contendo a conversão do pdf para html;
	- gera origem.db3 (conteúdo txt do pdf original para contruir o html) em res;
	- gera origem.jpg.db3 (imagens do pdf original para contruir o html) em res;
	- gera origem.idx.db3 (índice Full Text Search) em res;
	- copia o arquivo original origem.pdf que estava em orig para proc;

Ou seja, funciona de uma forma simples. O processamento é assim:
	- Há arquivos dentro de pastas AAMM dentro de orig? Há trabalho a ser feito;
	- Todo o processamento deve ser feito dentro das pastas _sistema\db3 e _sistema\tmp. Só depois de pronto deve ser jogado em proc ou res . 

 
Convenções

Tudo está em UTF-8.
	- Mas, atenção, o sistema de arquivos do Windows é todo em ISO-8859-1. Por isso, não custa nada TODA A VEZ que trabalhar com arquivos fazer o utf8_encode e utf_decode;





## Versão 4.2 - Fechar o frontent

### O projeto é o seguinte:
1. O usuário manda via um site de busca ainda não definido, como www.ppi.com.br, WhatsApp ou Telegram uma pergunta complexa e direta, como: “Como o juiz Zezinho da 7ª vara de São Paulo julga ações contendo Gafisa e Bem de Família?”
1. Em 5 minutos a até uma hora, no máximo, vem uma resposta simplesmente contendo os links de JusBrasil, Escavador e PublicoNet das publicações de Diário Oficial selecionadas
1. Cada usuário novo tem, por exemplo, 50 reais de crédito ‘grátis’. Na medida em que vai perguntando, o crédito vai diminuindo e ele tem que repor. Para isso, ele compra créditos no Mercado Livre. Ou pode ser feito o sistema uma pergunta grátis por semana.

### Como funciona?
É um “sistema de inteligência artificial avançado”, que “aprende na medida em que é questionado”
Na prática, este sistema PPI é operado por um humano, que faz todas as pesquisas e manda os links. AS pesquisas são mais complexas que as do jusbrasil, com mais opções, para o ser humano backend fazer. Com o passar dos tempos, vai automatizando o que é possível.
A parte de mostra de dados PublicoNet não pertence a um sócio conhecido – ele vive de pequenos anúncios, como CNPJ.info  . Isso porque ele pode sofrer processos porque reproduz conteúdo de outros. Nesse sentido, verificar a parte legal mais abaixo e fazer uma resposta padrão do tipo “não respondemos pesquisa sobre nome de pessoas ou empresas, sob o risco de divulgação de dados que possam causar prejuízo às mesmas”

### Ambiente

#### Quem é a concorrência?
- jusbrasil.com.br – tem assinatura – há ainda o jusbrasil pro (escritório online)
sobre custos: 
https://rafaelcosta.jusbrasil.com.br/artigos/454529208/o-jusbrasil-pode-acabar-em-6-meses-sim-definitivamente

- escavador.com (Potelo Sistemas de Informação Ltda ME) - tem assinatura
	Gratis	Pessoal	Profissional	Escritório
 Preço/Mês 	               -   	        9,90 	        29,90 	     119,90 
 Monitoramento de termos em Diários Oficiais 	1	3	10	50
 Monitoramento de Processos 	1	3	10	50
 Notificações/mês por monitoramento 	3	20	Ilimitado	Ilimitado
 Emails/Alertas 	Semana	Diário	3 emails	15 emails

- Radar Oficial www.radaroficial.com.br (Digesto Pesquisa e Banco de Dados Ltda) -– tem planos de 22 a 130 reais por mês
buscaoficial.com – não está funcionando
- www.arquivojudicial.com – Parece nem estar funcionando
- arquivobrasil.com – não está funcionando
- justotal.com – fora do ar. Tem uma petição reclamando dele aqui: https://peticaopublica.com.br/pview.aspx?pi=justotal

#### O que é lícito e ilícito?
É lícito fazer o que a Google e Jusbrasil fazem, ou seja, “disponibilizar os endereços, na internet, nos quais se encontra conteúdo postado por terceiros e que se encaixa nos parâmetros de pesquisa fornecidos pelo interessado.”
```
CF Art. 5º inc LX: “a lei só poderá restringir a publicidade dos atos processuais quando a defesa da intimidade ou o interesse social o exigirem;”
Art. 189 do CPC: “Os atos processuais são públicos, todavia tramitam em segredo de justiça os processos:
I - em que o exija o interesse público ou social;
II - que versem sobre casamento, separação de corpos, divórcio, separação, união estável, filiação, alimentos e guarda de crianças e adolescentes;
III - em que constem dados protegidos pelo direito constitucional à intimidade;
IV - que versem sobre arbitragem, inclusive sobre cumprimento de carta arbitral, desde que a confidencialidade estipulada na arbitragem seja comprovada perante o juízo.
§ 1o O direito de consultar os autos de processo que tramite em segredo de justiça e de pedir certidões de seus atos é restrito às partes e aos seus procuradores.
§ 2o O terceiro que demonstrar interesse jurídico pode requerer ao juiz certidão do dispositivo da sentença, bem como de inventário e de partilha resultantes de divórcio ou separação.”
Somente poderá haver sigilo processual se decretado pelo juiz responsável, na forma e nas hipóteses da lei. Inexistindo sigilo, a divulgação da existência dos processos e de seus atos e decisões não encontra nenhuma restrição legal. Inexistindo sigilo decretado pelo juiz, o processo é público, de modo que a divulgação de sua existência e de seu andamento, salvo abuso específico em caso concreto, não é ilegal.
```

```
CF Art. 220: 
“A manifestação do pensamento, a criação, a expressão e a informação, sob qualquer forma, processo ou veículo não sofrerão qualquer restrição, observado o disposto nesta Constituição.
§ 1º Nenhuma lei conterá dispositivo que possa constituir embaraço à plena liberdade de informação jornalística em qualquer veículo de comunicação social, observado o disposto no art. 5º, IV, V, X, XIII e XIV.
§ 2º É vedada toda e qualquer censura de natureza política, ideológica e artística.
§ 3º Compete à lei federal:
I - regular as diversões e espetáculos públicos, cabendo ao Poder Público informar sobre a natureza deles, as faixas etárias a que não se recomendem, locais e horários em que sua apresentação se mostre inadequada;
II - estabelecer os meios legais que garantam à pessoa e à família a possibilidade de se defenderem de programas ou programações de rádio e televisão que contrariem o disposto no art. 221, bem como da propaganda de produtos, práticas e serviços que possam ser nocivos à saúde e ao meio ambiente.
§ 4º A propaganda comercial de tabaco, bebidas alcoólicas, agrotóxicos, medicamentos e terapias estará sujeita a restrições legais, nos termos do inciso II do parágrafo anterior, e conterá, sempre que necessário, advertência sobre os malefícios decorrentes de seu uso.
§ 5º Os meios de comunicação social não podem, direta ou indiretamente, ser objeto de monopólio ou oligopólio.
§ 6º A publicação de veículo impresso de comunicação independe de licença de autoridade.”
```

```
A matéria já foi decidida em recurso especial pelo STJ:
“CIVIL E CONSUMIDOR. INTERNET. RELAÇÃO DE CONSUMO. INCIDÊNCIA DO CDC. GRATUIDADE DO SERVIÇO. INDIFERENÇA. PROVEDOR DE PESQUISA. FILTRAGEM PRÉVIA DAS BUSCAS. DESNECESSIDADE. RESTRIÇÃO DOS RESULTADOS. NÃO-CABIMENTO. CONTEÚDO PÚBLICO. DIREITO À INFORMAÇÃO. 1. A exploração comercial da Internet sujeita as relações de consumo daí advindas à Lei nº 8.078/90. 2. O fato de o serviço prestado pelo provedor de serviço de Internet ser gratuito não desvirtua a relação de consumo, pois o termo "mediante remuneração", contido no art. 3º, § 2º, do CDC, deve ser interpretado de forma ampla, de modo a incluir o ganho indireto do fornecedor. 3. O provedor de pesquisa é uma espécie do gênero provedor de conteúdo, pois não inclui, hospeda, organiza ou de qualquer outra forma gerencia as páginas virtuais indicadas nos resultados disponibilizados, se limitando a indicar links onde podem ser encontrados os termos ou expressões de busca fornecidos pelo próprio usuário. 4. A filtragem do conteúdo das pesquisas feitas por cada usuário não constitui atividade intrínseca ao serviço prestado pelos provedores de pesquisa, de modo que não se pode reputar defeituoso, nos termos do art. 14 do CDC, o site que não exerce esse controle sobre os resultados das buscas. 5. Os provedores de pesquisa realizam suas buscas dentro de um universo virtual, cujo acesso é público e irrestrito, ou seja, seu papel se restringe à identificação de páginas na web onde determinado dado ou informação, ainda que ilícito, estão sendo livremente veiculados. Dessa forma, ainda que seus mecanismos de busca facilitem o acesso e a consequente divulgação de páginas cujo conteúdo seja potencialmente ilegal, fato é que essas páginas são públicas e compõem a rede mundial de computadores e, por isso, aparecem no resultado dos sites de pesquisa. 6. Os provedores de pesquisa não podem ser obrigados a eliminar do seu sistema os resultados derivados da busca de determinado termo ou expressão, tampouco os resultados que apontem para uma foto ou texto específico, independentemente da indicação do URL da página onde este estiver inserido. 7. Não se pode, sob o pretexto de dificultar a propagação de conteúdo ilícito ou ofensivo na web, reprimir o direito da coletividade à informação. Sopesados os direitos envolvidos e o risco potencial de violação de cada um deles, o fiel da balança deve pender para a garantia da liberdade de informação assegurada pelo art. 220, § 1º, da CF/88, sobretudo considerando que a Internet representa, hoje, importante veículo de comunicação social de massa. 8. Preenchidos os requisitos indispensáveis à exclusão, da web, de uma determinada página virtual, sob a alegação de veicular conteúdo ilícito ou ofensivo - notadamente a identificação do URL dessa página - a vítima carecerá de interesse de agir contra o provedor de pesquisa, por absoluta falta de utilidade da jurisdição. Se a vítima identificou, via URL, o autor do ato ilícito, não tem motivo para demandar contra aquele que apenas facilita o acesso a esse ato que, até então, se encontra publicamente disponível na rede para divulgação. 9. Recurso especial provido (STJ - REsp:1316921 RJ 2011/0307909-6, Relator: Ministra NANCY ANDRIGHI, Data de Julgamento: 26/06/2012, T3 - TERCEIRA TURMA, Data de Publicação: DJe 29/06/2012”
```

#### Qual o Nome do Site?

- Já existe ppi.com.br, embora esteja desligado
Domínio ppi.com.br
 Titular: JAIRO A. C. DO C.
 Criado: 18/10/2017
 Expiração: 18/10/2018
 Status: Congelado
 Expiração: 18/10/2018

- Já existe ppibrasil.com.br – está ligado à http://www.ppiworldwide.com/
Domínio ppibrasil.com.br
 Titular: Promocoes Premier do Brasil Alphaville Ltda
 Criado: 03/08/2011 #8661636
 Expiração: 03/08/2021

- Em suma, não sei o que fazer ainda com nome do site... deixar mais para frente
