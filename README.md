# Postman + Newman | APIs públicas do Brasil

[![Testes de API](https://github.com/joaovictorjesus204/postman-newman-apis-brasil/actions/workflows/newman.yml/badge.svg)](https://github.com/joaovictorjesus204/postman-newman-apis-brasil/actions/workflows/newman.yml)

**📊 Relatório da última execução:** https://joaovictorjesus204.github.io/postman-newman-apis-brasil/ (bancadas: [bancadas.html](https://joaovictorjesus204.github.io/postman-newman-apis-brasil/bancadas.html))

Testes de API com **Postman** e **Newman** em três APIs públicas brasileiras:

| API | O que é | O que é testado |
|---|---|---|
| **[ViaCEP](https://viacep.com.br)** | Consulta de CEP usada por lojas e cadastros do país inteiro | CEP válido, inválido e inexistente, busca por endereço e resposta em XML |
| **[Câmara dos Deputados](https://dadosabertos.camara.leg.br/swagger/api.html)** | Dados Abertos da Câmara (v2) | Filtro, ordenação, paginação, encadeamento lista → detalhe e bancadas por estado |
| **[Banco Central (PTAX)](https://olinda.bcb.gov.br/olinda/servico/PTAX/versao/v1/aplicacao)** | Cotação oficial do dólar e do euro | Período, consulta por dia, fim de semana e coerência entre endpoints |

Roda no Postman (com interface) ou no terminal com Newman, e em CI no **GitHub Actions** e no **Azure DevOps**.

Autor: João Victor de Jesus

---

## Números

| | |
|---|---|
| Requisições | 15 na coleção principal + 27 iterações orientadas a dados (uma por estado) |
| Verificações | 192 (57 na coleção principal e 135 nas bancadas) |
| Contrato | JSON Schema no ViaCEP, na Câmara e no Banco Central |
| Encadeamento | Lista de deputados → detalhe → próxima página; dólar no período → mesmo dia na consulta diária |
| Execução automática | A cada push e diariamente, nos dias úteis, pelo GitHub Actions |
| Relatórios | HTML (htmlextra) publicado no GitHub Pages e JUnit para o Azure DevOps |

---

## Como rodar

Pré-requisito: **Node.js 20+**.

```bash
git clone https://github.com/joaovictorjesus204/postman-newman-apis-brasil.git
cd postman-newman-apis-brasil
npm install
npm test
```

Relatórios: `reports\report.html` (coleção principal) e `reports\bancadas.html` (bancadas).

| Comando | O que faz |
|---|---|
| `npm test` | Roda as duas coleções e gera os relatórios HTML e JUnit |
| `npm run test:ci` | O mesmo, pulando a Câmara (usado no CI, fora do Brasil) |
| `npm run test:apis` | Só a coleção principal |
| `npm run test:bancadas` | Só as bancadas, com `data/bancadas.csv` |
| `npm run test:viacep` / `test:camara` / `test:bcb` | Só uma API |

### Ver os testes rodando

| Como | O que mostra |
|---|---|
| `npm run test:verbose` | No terminal, cada requisição com cabeçalhos, corpo e tempo de resposta |
| **No Postman** | Importe `postman/apis-brasil.postman_collection.json` e `postman/ambiente.postman_environment.json`, escolha o ambiente "APIs públicas do Brasil" no canto superior direito e clique em **Run** na coleção |
| **Bancadas no Postman** | Importe `postman/bancadas.postman_collection.json`, clique em **Run**, em **Select File** escolha `data/bancadas.csv` e rode |

---

## Cobertura

| ID | Cenário | Tipo |
|---|---|---|
| CEP-01 | CEP válido retorna o endereço completo, com contrato e código IBGE | Positivo, contrato |
| CEP-02 | CEP com formato inválido é recusado com 400 | Negativo |
| CEP-03 | CEP inexistente informa erro no corpo | Negativo |
| CEP-04 | Busca por endereço retorna só ruas da cidade pedida | Filtro |
| CEP-05 | Mesmo CEP em XML traz os mesmos dados do JSON | Formato alternativo |
| CAM-01 | Deputados de SP: limite por página, filtro por UF, ordem alfabética e link de paginação | Filtro, ordenação, contrato |
| CAM-02 | Detalhe do deputado escolhido na lista | Encadeamento |
| CAM-03 | Próxima página não repete deputados e continua a ordem | Paginação |
| CAM-04 | Partidos com id, sigla e nome, sem sigla repetida | Integridade |
| CAM-05 | Deputado inexistente retorna erro do cliente (4xx) | Negativo |
| BAN-01 | Cada estado tem o número de cadeiras da Constituição (27 iterações) | Orientado a dados, regra de negócio |
| BCB-01 | Moedas disponíveis incluem dólar e euro | Referência |
| BCB-02 | Dólar nos últimos 30 dias: contrato, dias úteis, venda ≥ compra, valores plausíveis | Regra de negócio, contrato |
| BCB-03 | Consulta por dia confere com a consulta por período | Encadeamento, consistência |
| BCB-04 | Domingo não tem cotação | Negativo |
| BCB-05 | Euro de fechamento acima do dólar | Regra de negócio |
| Todas | Resposta em até 5 segundos (15 s na Câmara, ver Achados) | Desempenho |

---

## Estrutura

```
postman/apis-brasil.postman_collection.json     coleção principal (ViaCEP, Câmara, Banco Central)
postman/bancadas.postman_collection.json        coleção orientada a dados
postman/ambiente.postman_environment.json       endereços das 3 APIs
data/bancadas.csv                               27 estados e o número de cadeiras de cada um
.github/workflows/newman.yml                    CI no GitHub Actions com relatório no GitHub Pages
azure-pipelines.yml                             o mesmo pipeline no Azure DevOps
```

---

## Decisões técnicas

| Decisão | Por quê |
|---|---|
| **Coleção versionada em JSON** | A coleção exportada do Postman fica no Git, com histórico e revisão como qualquer código |
| **Ambiente separado** (`ambiente.postman_environment.json`) | As URLs ficam fora dos testes. Trocar para homologação é trocar o ambiente |
| **Newman no terminal e no CI** | A mesma coleção roda no Postman, no terminal e no pipeline, sem reescrever nada |
| **JSON Schema** (`pm.response.to.have.jsonSchema`) | Valida contrato: campos obrigatórios, tipos e formatos (CEP com hífen, código IBGE com 7 dígitos) |
| **Encadeamento por variáveis da coleção** | A lista de deputados define quem é detalhado, e o período do dólar define o dia consultado. Os testes usam dados reais do momento, sem id fixo |
| **Datas calculadas no pre-request** | O período do dólar são sempre os últimos 30 dias, e o domingo é sempre o último domingo. A coleção não envelhece |
| **Bancadas orientadas a dados** (`-d data/bancadas.csv`) | Um teste, 27 estados. A regra vem da Constituição (mínimo 8, máximo 70 cadeiras), e não do que a API devolve |
| **Validação de regra, não de valor do dia** | A cotação muda todo dia. Os testes conferem que a venda é maior ou igual à compra, que o euro vale mais que o dólar e que os dois endpoints dão o mesmo valor |
| **Nova tentativa em erro 5xx** (`pm.execution.setNextRequest`) | Na segunda execução local, a API do Banco Central respondeu `502 Bad Gateway` uma vez, e na execução anterior tinha funcionado. Agora a coleção repete a requisição até 2 vezes, com 3 s de pausa, e registra o aviso no log. Erro que persiste continua reprovando |
| **Câmara pulada no CI** (`pm.execution.skipRequest`) | No GitHub Actions, a API da Câmara respondeu `504 upstream request timeout` em quase todas as requisições, enquanto ViaCEP e Banco Central responderam normalmente. No computador local ela também teve períodos de lentidão e 504, mas em menor proporção (ver Achados). Para o CI não ficar vermelho por instabilidade de um serviço de terceiro, no `npm run test:ci` as requisições da Câmara aparecem como puladas, com o motivo no log. No computador local, `npm test` roda tudo, com nova tentativa em 5xx |
| **Teste de tempo de resposta na coleção** | Escrito uma vez, vale para todas as requisições |
| **Relatórios htmlextra e JUnit** | O HTML é legível por quem não é desenvolvedor. O JUnit alimenta a aba Tests do Azure DevOps |
| **`run-s --continue-on-error`** | Se a coleção principal falhar, as bancadas rodam mesmo assim e o resultado mostra tudo |
| **Pipeline também no Azure DevOps** | O `azure-pipelines.yml` publica os resultados na aba Tests e guarda os relatórios como artefato |

---

## Achados

| Achado | Detalhe |
|---|---|
| **ViaCEP devolve o XML como `application/xhtml+xml`** | Esse tipo indica página web (XHTML). Para dado em XML, o esperado é `application/xml` ou `text/xml`. Um cliente que decide como ler a resposta pelo Content-Type pode tratar o retorno como página. O CEP-05 aceita o tipo atual e registra o aviso no log |
| **API da Câmara lenta e com 504 intermitente** | No mesmo dia, do mesmo computador, a listagem de deputados respondeu em 50 ms numa execução e entre 8 e 15 s em outra, com `504 upstream request timeout` em 8 de 27 estados. O 504 chega sempre em 15 s, o que indica o tempo limite do gateway da Câmara esperando o servidor interno. A coleção repete a requisição em 5xx e usa 15 s como limite de tempo para a Câmara; para as outras APIs o limite continua 5 s |
| **ViaCEP responde 200 para CEP inexistente** | O erro vem no corpo (`"erro": "true"`). O esperado no padrão REST seria 404. O teste CEP-03 valida o comportamento atual e aceita o campo como texto ou booleano, porque a API já mudou esse tipo antes |

## Limitações

- APIs públicas de terceiros: instabilidade ou mudança de contrato podem quebrar a execução. A execução diária detecta isso rápido.
- A API da Câmara fica instável com frequência, e no GitHub Actions e no Azure DevOps (servidores no exterior) responde 504 em quase todas as requisições. Nesses ambientes, os testes da Câmara e das bancadas ficam como pulados; eles rodam completos no computador local.
- O número de deputados em exercício pode variar por uma vaga momentânea, por isso BAN-01 aceita uma cadeira a menos.

## Próximos passos

- Mock server do Postman para testar cenários de erro que as APIs reais não permitem forçar (500, timeout).
- Monitor do Postman agendado, com alerta por e-mail.
- Testes de segurança básicos: cabeçalhos HTTP e HTTPS obrigatório.
