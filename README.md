# Trabalho-performance-API-Pets-John-Pollette_Pgats2026

Autor: John Pollette
Data: 23 de Novembro de 2026

1. Objetivo

Avaliar o desempenho da API de Pets utilizando o Apache JMeter, comparando o comportamento da aplicação sob dois modelos de carga:

Modelo fechado (Thread Group): número fixo de usuários virtuais.
Modelo aberto (Open Model Thread Group): taxa de chegada constante de usuários.
2. API testada

API local, executada em http://localhost:3000, com documentação Swagger em http://localhost:3000/api-docs.

Endpoints testados (ambos exigem autenticação via token Bearer):

Método	Endpoint	Descrição	Resposta esperada
POST	/pets	Cadastra um pet para o usuário	201
GET	/pets	Consulta todos os pets	200

Antes de executar as consultas, o teste realiza o login e armazena o token para uso nas requisições seguintes.

3. Pré-requisitos
Java JDK 8 ou superior
Apache JMeter 5.5 ou superior
API em execução em localhost:3000
4. Estrutura do plano de testes
Test Plan
├── User Defined Variables
├── View Results Tree
├── setUp Thread Group           → Login e extração do token
├── Thread Group                 → Modelo FECHADO
│   ├── HTTP Header Manager      (Authorization: Bearer <token>)
│   ├── POST /pets
│   └── GET /pets
├── Open Model Thread Group      → Modelo ABERTO
│   ├── HTTP Header Manager      (Authorization: Bearer <token>)
│   ├── POST /pets
│   └── GET /pets
└── Summary Report

Autenticação

Um setUp Thread Group executa o login uma única vez. O token é extraído da resposta com um JSON Extractor e compartilhado com os demais Thread Groups por meio de uma propriedade do JMeter (props), sendo enviado no cabeçalho Authorization: Bearer <token>.

Dados de teste

O corpo do POST /pets usa nomes únicos gerados com ${__UUID()}, pois a API retorna 409 quando o pet já existe para o usuário:

json
{"name": "Pet_${__UUID()}", "type": "Cão"}
Validações

Cada requisição possui uma Response Assertion que valida o código HTTP (201 para o POST e 200 para o GET).

5. Cenários de carga
5.1 Modelo fechado
Parâmetro	Valor
Número de threads (usuários)	20
Ramp-up	20 segundos
Loop count	10
Requisições por iteração	POST /pets + GET /pets
Total estimado	400 requisições (200 POST + 200 GET)

Característica: cada usuário só envia a próxima requisição depois de receber a resposta da anterior. Se a API ficar lenta, a taxa de requisições diminui.

5.2 Modelo aberto
Parâmetro	Valor
Schedule	rate(1/sec) random_arrival(10 sec) rate(5/sec) random_arrival(40 sec) rate(5/sec)
Subida	10 s (de 1 para 5 usuários/s)
Carga estável	40 s a 5 usuários/s
Requisições por iteração	POST /pets + GET /pets
Total estimado	~225 iterações

Característica: os usuários chegam a uma taxa definida, independentemente do tempo de resposta da API. Se a API ficar lenta, as requisições se acumulam.

6. Como executar
Pela interface gráfica (para validar)
Abra o JMeter e carregue o arquivo teste-pets.jmx.
Ajuste, se necessário, as credenciais de login e o endpoint de autenticação.
Clique em Start (botão verde) e acompanhe os resultados nos listeners.
Pela linha de comando (execução final)
bash
jmeter -n -t John_Pollette_PGATS_2026_Trabalho.jmx -l logs.jtl -e -o relatorio
-n: modo não gráfico
-t: arquivo do plano de testes
-l: arquivo de resultados
-e -o relatorio: gera o relatório HTML na pasta relatorio (a pasta deve estar vazia ou não existir)
