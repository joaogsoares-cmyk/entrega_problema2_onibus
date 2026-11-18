# Problema 2 — A que horas passa o próximo ônibus?

**A nova funcionalidade — Problema 2**  
Universidade do Distrito Federal (UnDF) · Ciência da Computação · Prof.ª Carolina Gonçalves Abreu  
**Autor indicado no modelo:** João Victor Gomes Soares  
**Data:** novembro de 2026  

> **Nota de reprodutibilidade e histórico:** o arquivo curto, as seis cargas oficiais e seus gabaritos foram recebidos e usados. Tempos e memória foram medidos no sandbox cloud descrito na Seção 2.1, não no computador do grupo. O grupo informou que ainda não tem commits, e os registros semanais do redator não foram disponibilizados; por isso, sessões e hashes históricos não podem ser atribuídos. Essa limitação está registrada na tabela de decisões, sem criação de evidências retroativas.

## Resumo

Este trabalho implementa um sistema de registro e consulta de chegadas de ônibus, comparando uma árvore binária de busca (ABB) sem balanceamento e uma árvore AVL. A chave ordena cada chegada pelo instante cronológico e, em caso de empate, pelo código da linha. A primeira versão suporta as oito operações solicitadas. Na terceira fase, uma AVL principal mantém todas as chegadas e um índice auxiliar composto por AVL de linhas e AVL temporal por linha acelera as consultas específicas de linha e as esperas semanais. Nas cargas oficiais, a ABB quase ordenada chegou à altura 17.750 em N=20.000, enquanto a AVL ficou com altura 15; os tempos medianos foram 10.017,246 ms e 232,539 ms, respectivamente, neste sandbox cloud. **Palavras-chave:** árvore binária de busca; árvore AVL; transporte público; análise de algoritmos.

## 1. Introdução

O deslocamento de estudantes à Universidade do Distrito Federal depende, em muitos casos, do transporte público. Avisos sobre a passagem dos ônibus em grupos de mensagens podem se perder entre conversas, e previsões de aplicativos nem sempre coincidem com o que ocorreu. Um registro cronológico consultável pode responder perguntas como se uma linha já passou desde determinado horário, quanto tempo houve de espera e quais ônibus chegaram em um intervalo.

O objetivo é implementar as operações de registro, busca, próximo ônibus, consulta por linha, intervalo, remoção, altura e espera semanal, e estudar o custo de duas estruturas: uma ABB implementada sem balanceamento e uma AVL desenvolvida pelo grupo. A hipótese é que a ABB pode ser eficiente enquanto a ordem de inserção é favorável, mas pode degenerar com avisos quase cronológicos; a AVL limita a altura e oferece garantias logarítmicas para as operações de busca e atualização.

O sistema organiza evidências observadas, mas não prova por si só a regularidade futura do serviço nem substitui validação independente dos avisos. A discussão final delimita o que os dados permitem afirmar.

## 2. Materiais e métodos

### 2.1 Ambiente e reprodutibilidade

O programa foi escrito em Python 3.12.3 e não requer dependências de terceiros. As medições apresentadas foram realizadas em sandbox cloud Linux x86_64, kernel 6.18.38, 6 vCPUs reportadas, CPU AMD EPYC e 23 GiB de memória total reportada. Esse ambiente não é a máquina local do grupo. A pasta contém `executar.txt` com os comandos da Fase 1 e da Fase 3 e um README com comandos para execução e geração das tabelas. Foram usados os seis arquivos oficiais e o arquivo pequeno oficial; os casos sintéticos adicionais servem somente para teste técnico.

Os dados de cada chegada são convertidos para um inteiro de segundos: ordinal da data multiplicado por 86.400, somado aos segundos desde meia-noite. A chave completa é `(instante_em_segundos, código_da_linha)`. Assim, a ordem principal é temporal e empates no instante são desfeitos comparando os códigos caractere a caractere. Como uma mesma linha não pode ter duas chegadas no mesmo instante, a chave também identifica unicamente um registro válido. Datas são convertidas pela biblioteca padrão, sem estruturas prontas de ordenação.

### 2.2 Fase 1: árvore binária de busca

A ABB é implementada com nós próprios e ligações esquerda/direita. A inserção e busca seguem a comparação da chave composta. R3 localiza o menor registro cuja chave não seja anterior ao instante consultado. R5 percorre em ordem apenas o sufixo a partir do início do intervalo e interrompe ao passar do fim. Na remoção de nó com dois filhos, o valor é substituído pelo sucessor em ordem (menor chave da subárvore direita), que é então removido de sua posição original. A altura é calculada por percurso iterativo, com altura zero para árvore vazia e um para nó único.

Para R4 e R8, nesta fase, não há índice por linha: a ABB é percorrida em ordem a partir do instante consultado, filtrando os registros da linha e do dia. A decisão evita memória auxiliar, mas pode examinar uma fração grande ou todos os registros. R8 executa essa busca para cada uma das k semanas anteriores, da mais antiga à data informada, e devolve diferença em segundos ou `NAOPASSOU`.

### 2.3 Fase 3: árvores AVL e índice auxiliar

A árvore principal é uma AVL própria com fator de balanceamento baseado nas alturas dos filhos, rotações simples e duplas, e rebalanceamento recursivo após inserção e remoção. A remoção usa sucessor em ordem; após retirar o sucessor, atualiza alturas e rebalanceia o caminho até a raiz. A altura armazenada no nó permite responder ALT em tempo constante.

Para R4 e R8, foi escolhido um índice auxiliar formado por uma AVL de códigos de linha; cada nó armazena uma AVL dos instantes daquela linha. A primeira AVL encontra a linha, e a segunda encontra por busca de limite inferior a chegada mais próxima a partir do instante pedido. Para R8, essa consulta se repete para cada data semanal. O índice é atualizado somente após uma inserção/remoção bem-sucedida na árvore principal; chegada duplicada ou remoção inexistente não altera o índice. Ao apagar a última chegada de uma linha, o nó daquela linha é removido do índice.

### 2.4 Decisões de projeto

| Decisão | Alternativa descartada | Motivo da escolha | Sessão do registro | Commit de implementação |
|---|---|---|---|---|
| Chave `(instante inteiro, linha)` | Texto único como `19_10_66600` | Preserva ordem cronológica e desempate de linhas no mesmo instante, com identificação sem ambiguidade. | Registro semanal não fornecido; sessão não confirmada | Não há commit: o grupo informou que ainda não tem commits |
| Remoção AVL pelo sucessor em ordem | Predecessor em ordem ou outra estratégia válida | Reutiliza a convenção da Fase 1 e mantém a propriedade de busca; exige rebalancear o caminho afetado. | Registro semanal não fornecido; sessão não confirmada | Não há commit: o grupo informou que ainda não tem commits |
| R4/R8 por índice AVL de linhas e AVL temporal | Percorrer a árvore principal inteira para cada consulta | Reduz a busca específica de linha a custos logarítmicos, pagando por nós duplicados e manutenção sincronizada. | Registro semanal não fornecido; sessão não confirmada | Não há commit: o grupo informou que ainda não tem commits |

**Limitação documental:** as decisões técnicas e suas justificativas estão descritas acima, mas não foi possível vinculá-las a sessões específicas porque os registros do redator não foram fornecidos. O grupo informou que ainda não existem commits. Assim, não há hashes a citar nem histórico de desenvolvimento a auditar; as decisões aqui registradas descrevem a implementação entregue, não evidenciam em que sessão cada uma foi tomada.

### 2.5 Procedimento de medição

Para cada um dos seis arquivos oficiais foram realizadas cinco execuções de cada fase. Cada arquivo contém N comandos `REG` e também operações de consulta, remoção e altura; portanto o tempo da tabela é do arquivo inteiro, não somente da inserção. `medir_arquivo.py` imprime os tempos individuais, a mediana e a altura após uma execução separada de `ALT`. O programa lê o arquivo e separa os campos antes de iniciar o cronômetro. Validação das operações, conversão de datas/horas, processamento e montagem das respostas estão incluídos no tempo `TEMPO_MS`; a leitura inicial e a escrita final das respostas ficam fora da medição.

A memória foi estimada por `tracemalloc` no estado final após aplicar todos os `REG` e `REM`. A medida exclui interpretador, leitura/parsing/lista de operações e saída; reporta alocação retida, em MiB (bytes/1.048.576), e não equivale ao RSS completo. Os valores abaixo pertencem ao sandbox cloud da Seção 2.1; no computador do grupo os comandos reproduzíveis estão no README.

## 3. Resultados

### 3.1 Fase 2: ABB sem balanceamento

**Tabela 1 — Mediana do tempo e altura final da ABB nos arquivos oficiais.** Tempo em milissegundos, altura em níveis. N é a quantidade de comandos `REG` do arquivo.

| N | Ordem dos registros | Mediana (ms), 5 execuções | Altura final |
|---:|---|---:|---:|
| 1.000 | Quase em ordem | 27,132 | 889 |
| 5.000 | Quase em ordem | 618,620 | 4.458 |
| 20.000 | Quase em ordem | 10.017,246 | 17.750 |
| 1.000 | Embaralhados | 3,112 | 22 |
| 5.000 | Embaralhados | 18,550 | 30 |
| 20.000 | Embaralhados | 146,683 | 32 |

Para a ABB, a razão entre medianas de N=20.000 e N=1.000 foi 369,204 na sequência quase ordenada e 47,135 na embaralhada. A primeira cresce aproximadamente como o aumento da altura (de 889 para 17.750); na embaralhada, a altura só passa de 22 para 32 e o tempo cresce bem menos. Isso evidencia a degeneração da ABB perante a ordem quase cronológica deste arquivo. Em sequência estritamente crescente, a forma-limite é uma cadeia à direita:

```text
v1
└── v2
    └── v3
        └── v4
            └── v5
                └── v6
                    └── v7
```

Nesse arquivo, as sete primeiras chegadas foram inseridas na seguinte ordem: 18:22:10 (136.1), 18:31:45 (0.603), 18:40:00 (136.2), 18:40:00 (136.1), 18:55:20 (136.1), 19:10:05 (603.1) e 18:27:30 (136.3). A ABB resultante tem altura 5; as arestas indicam filho esquerdo/direito. Em sequência estritamente crescente, ao contrário desta amostra com uma chegada atrasada, a ABB vira uma cadeia e a busca pode custar Θ(n).

**Figura 1 — ABB após as sete primeiras chegadas do arquivo curto oficial.**

![ABB real após sete registros do arquivo pequeno; rótulos apresentam data, horário e linha.](https://private-us-east-1.manuscdn.com/sessionFile/TrENXdsHCd925jbrGwSBls/sandbox/stnEtQEhtNaNXpLb5Kcs0u-images_1794954654169_na1fn_L2hvbWUvdWJ1bnR1L2VudHJlZ2Ffb25pYnVzL3JlbGF0b3Jpby9maWd1cmFzL2FiYl9wcmltZWlyYXNfc2V0ZQ.png?Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9wcml2YXRlLXVzLWVhc3QtMS5tYW51c2Nkbi5jb20vc2Vzc2lvbkZpbGUvVHJFTlhkc0hDZDkyNWpickd3U0Jscy9zYW5kYm94L3N0bkV0UUVodE5hTlhwTGI1S2NzMHUtaW1hZ2VzXzE3OTQ5NTQ2NTQxNjlfbmExZm5fTDJodmJXVXZkV0oxYm5SMUwyVnVkSEpsWjJGZmIyNXBZblZ6TDNKbGJHRjBiM0pwYnk5bWFXZDFjbUZ6TDJGaVlsOXdjbWx0WldseVlYTmZjMlYwWlEucG5nIiwiQ29uZGl0aW9uIjp7IkRhdGVMZXNzVGhhbiI6eyJBV1M6RXBvY2hUaW1lIjoxNzk3MjkyODAwfX19XX0_&Key-Pair-Id=K2QY5QTL8JSY6C&Signature=MEUCIQDTtuHWBUHAZmTsI3nqDKhQh3x-36vxqmQ2yK01yxcQxAIgXdmJng0O0rB36wc3mndy2dOuEIXX6RYj5H-LVFIJOBk_)

### 3.2 Fase 3: AVL

A AVL assegura altura O(log n) para toda sequência de inserções/remoções. A condição indicada no enunciado, `altura ≤ 1,45 × log₂(n + 2)`, deve ser conferida com o número efetivo de nós após cada carga:

| Carga oficial | n final | Altura observada | Limite `1,45 × log₂(n + 2)` | Respeita? |
|---|---:|---:|---:|---|
| N=1.000, quase em ordem | 987 | 11 | 14,427 | Sim |
| N=5.000, quase em ordem | 4.961 | 13 | 17,802 | Sim |
| N=20.000, quase em ordem | 19.825 | 15 | 20,699 | Sim |
| N=1.000, embaralhado | 987 | 12 | 14,427 | Sim |
| N=5.000, embaralhado | 4.961 | 15 | 17,802 | Sim |
| N=20.000, embaralhado | 19.825 | 17 | 20,699 | Sim |

Ao inserir a terceira chegada do arquivo curto (`2026-10-05 18:40:00 136.2`), a AVL observa o caso RR no nó `A` (`2026-10-05 18:22:10 136.1`) e aplica rotação simples à esquerda. A chave `B` (`18:31:45 0.603`) passa a raiz desse trecho, com `A` à esquerda e `C` (`18:40:00 136.2`) à direita.

**Figura 2 — Primeira rotação AVL observada no arquivo curto: caso RR e rotação simples à esquerda.**

![Rotação RR real após a terceira chegada do arquivo curto, antes e depois.](https://private-us-east-1.manuscdn.com/sessionFile/TrENXdsHCd925jbrGwSBls/sandbox/stnEtQEhtNaNXpLb5Kcs0u-images_1794954654169_na1fn_L2hvbWUvdWJ1bnR1L2VudHJlZ2Ffb25pYnVzL3JlbGF0b3Jpby9maWd1cmFzL3JvdGFjYW9fYXZs.png?Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9wcml2YXRlLXVzLWVhc3QtMS5tYW51c2Nkbi5jb20vc2Vzc2lvbkZpbGUvVHJFTlhkc0hDZDkyNWpickd3U0Jscy9zYW5kYm94L3N0bkV0UUVodE5hTlhwTGI1S2NzMHUtaW1hZ2VzXzE3OTQ5NTQ2NTQxNjlfbmExZm5fTDJodmJXVXZkV0oxYm5SMUwyVnVkSEpsWjJGZmIyNXBZblZ6TDNKbGJHRjBiM0pwYnk5bWFXZDFjbUZ6TDNKdmRHRmpZVzlmWVhacy5wbmciLCJDb25kaXRpb24iOnsiRGF0ZUxlc3NUaGFuIjp7IkFXUzpFcG9jaFRpbWUiOjE3OTcyOTI4MDB9fX1dfQ__&Key-Pair-Id=K2QY5QTL8JSY6C&Signature=MEUCIQCYMx5Pfy2G5HBzB1sT1mVhL-esKZHvPamXQJtGXPL8fQIgfHsyaUr8COy5QgROnmfIADCGUfg1r9EXwQ8JKMNky30_)

**Tabela 2 — Comparação lado a lado das Fases 1 e 3.** Memória retida em MiB (`tracemalloc`) após REG e REM.

| Arquivo oficial | Tempo F1 (ms) | Tempo F3 (ms) | Altura F1 | Altura F3 | Memória F1 | Memória F3 |
|---|---:|---:|---:|---:|---:|---:|
| N=1.000, quase em ordem | 27,132 | 7,523 | 889 | 11 | 0,140 | 0,226 |
| N=5.000, quase em ordem | 618,620 | 46,067 | 4.458 | 13 | 0,701 | 1,120 |
| N=20.000, quase em ordem | 10.017,246 | 232,539 | 17.750 | 15 | 2,799 | 4,466 |
| N=1.000, embaralhado | 3,112 | 7,271 | 22 | 12 | 0,140 | 0,226 |
| N=5.000, embaralhado | 18,550 | 42,447 | 30 | 15 | 0,701 | 1,120 |
| N=20.000, embaralhado | 146,683 | 212,057 | 32 | 17 | 2,799 | 4,466 |

O número final de nós ativos foi 987, 4.961 e 19.825 para N=1.000, 5.000 e 20.000, respectivamente, após aplicar as remoções. Foi igual nas duas ordens e nas duas fases.

**Tabela 3 — Verificação do limite de altura AVL.** O limite deve ser recalculado usando n da árvore ao final da execução, sem contar chegadas removidas.

| Arquivo | n | `1,45 × log₂(n + 2)` | Altura | Resultado |
|---|---:|---:|---:|---|
| N=1.000, quase em ordem | 987 | 14,427 | 11 | Sim |
| N=5.000, quase em ordem | 4.961 | 17,802 | 13 | Sim |
| N=20.000, quase em ordem | 19.825 | 20,699 | 15 | Sim |
| N=1.000, embaralhado | 987 | 14,427 | 12 | Sim |
| N=5.000, embaralhado | 4.961 | 17,802 | 15 | Sim |
| N=20.000, embaralhado | 19.825 | 20,699 | 17 | Sim |

### 3.3 Custo de R4 e R8 e memória

Com uma única árvore ordenada por instante, R4 localiza um limite inferior e percorre os registros seguintes até achar a linha. Seu custo é O(h+s), em que s é o número de registros examinados; no pior caso, s=Θ(n), por exemplo quando a linha não volta a aparecer. Esse custo pode crescer com os meses de histórico. R8 é diferente: a consulta é limitada ao dia de cada semana. Sem índice por linha, o custo é O(kh + Σdᵢ), onde h é a altura e dᵢ é o número de chegadas daquele dia examinadas. O pior caso continua O(kn), mas o histórico inteiro não é necessariamente varrido em cada semana; se o número de chegadas por dia for estável, o termo de varredura por dia também tende a se manter estável. Assim, R8 pode dispensar um índice de linha se medições e requisitos aceitarem esse custo por dia.

Com o índice AVL implementado, R4 custa O(log ℓ + log n_l), sendo ℓ a quantidade de linhas e n_l a quantidade de registros da linha consultada. R8 custa O(k(log ℓ + log n_l)). Mantidos fixos ℓ e k, aumentar o histórico da linha aumenta o custo logaritmicamente, em vez de linearmente. Se as consultas abrangerem mais semanas, o custo cresce linearmente em k.

O custo espacial é O(n + ℓ) nós adicionais além da árvore principal: há uma árvore de timestamps contendo cada chegada novamente, e uma AVL de índice com até um nó por linha. Toda inserção válida atualiza as duas árvores e, se preciso, cria o nó da linha; remoção válida atualiza as duas árvores e remove a linha quando fica vazia. Na medição oficial, para n final=19.825, a memória retida foi 2,799 MiB na Fase 1 e 4,468 MiB na Fase 3, cerca de 1,60×. O índice paga memória e manutenção adicional para reduzir buscas por linha.

Para isolar as consultas, foi construída a estrutura final aplicando os REG e REM do arquivo quase ordenado e cronometradas 500 chamadas repetidas, em cinco rodadas; construção e parsing ficaram fora do cronômetro. Usou-se a linha ausente `ZZZZ`, um caso de varredura desfavorável para a ABB. Cada `SEMANA` foi configurada com k=4; os valores da última coluna são por chamada completa SEMANA, não por dia individual. No ensaio “AVL principal sem índice”, foi construída apenas a AVL das chegadas, e R8 percorre em ordem a janela do dia e filtra a linha solicitada.

| Operação e fase | Registros ativos | Mediana total das 500 chamadas (ms) | Média por chamada (ms) |
|---|---:|---:|---:|
| PROXL, ABB, N=1.000 | 987 | 29,595 | 0,059190 |
| PROXL, ABB, N=20.000 | 19.825 | 615,528 | 1,231055 |
| PROXL, AVL + índice, N=1.000 | 987 | 0,066 | 0,000132 |
| PROXL, AVL + índice, N=20.000 | 19.825 | 0,068 | 0,000136 |
| SEMANA, ABB, N=1.000 (k=4) | 987 | 1,707 | 0,003414 |
| SEMANA, ABB, N=20.000 (k=4) | 19.825 | 6,564 | 0,013128 |
| SEMANA, AVL principal sem índice, N=1.000 (k=4) | 987 | 2,473 | 0,004946 |
| SEMANA, AVL principal sem índice, N=20.000 (k=4) | 19.825 | 7,091 | 0,014182 |
| SEMANA, AVL + índice, N=1.000 (k=4) | 987 | 0,358 | 0,000716 |
| SEMANA, AVL + índice, N=20.000 (k=4) | 19.825 | 0,362 | 0,000724 |

Neste pior caso, o custo ABB de PROXL subiu aproximadamente 20,8× entre 987 e 19.825 registros, próximo ao aumento do tamanho, enquanto a consulta indexada ficou praticamente constante. Para demonstrar que R8 pode ser resolvida sem árvore auxiliar de linha, foi também percorrida apenas a janela do dia na AVL principal. A chamada SEMANA sem índice passou de 0,004946 ms para 0,014182 ms (2,87×, ante crescimento de 20,1× no número de nós); com índice, ficou em 0,000716–0,000724 ms. Portanto, a janela diária limitada torna viável implementar R8 sem índice por linha para o volume observado, ao custo de examinar as chegadas daquele dia. O código mantém o índice compartilhado porque ele já é necessário para R4 e também reduz o tempo de R8 (neste ensaio, por cerca de 6,9× em N=1.000 e 19,6× em N=20.000), pagando memória e sincronização adicionais. Isso explicita a decisão e não afirma que a contagem diária será sempre constante.

### 3.4 Complexidade das operações

Seja n o número de chegadas ativas; h a altura da ABB; ℓ o número de linhas; n_l os registros da linha consultada; r o número de resultados de ENTRE; k o número de semanas consultadas. Custos de saída não podem ser inferiores ao tamanho da resposta.

| Operação | Fase 1 — ABB | Garantia Fase 1 | Fase 3 — AVL e índice | Garantia Fase 3 |
|---|---|---|---|---|
| REG | O(h) | pior Θ(n), média esperada O(log n) sob ordens aleatórias | O(log n + log ℓ + log n_l) | pior O(log n + log ℓ + log n_l) |
| BUSCA | O(h) | pior Θ(n) | O(log n) | pior O(log n) |
| PROX | O(h) para limite inferior | pior Θ(n) | O(log n) | pior O(log n) |
| PROXL | O(n) | pior Θ(n) | O(log ℓ + log n_l) | pior O(log ℓ + log n_l) |
| ENTRE | O(h + r) percorrendo em ordem | pior O(n) | O(log n + r) | pior O(log n + r) |
| REM | O(h), mais busca do sucessor quando necessário | pior Θ(n) | O(log n + log ℓ + log n_l) para manter índice | pior O(log n + log ℓ + log n_l) |
| ALT | O(n) nesta implementação iterativa | Θ(n) | O(1), altura guardada na raiz | Θ(1) |
| SEMANA | O(k n) no pior caso | pior Θ(k n) | O(k(log ℓ + log n_l)) | pior O(k(log ℓ + log n_l)) |

Na ABB, h pode variar de Θ(log n) em formatos favoráveis até n em árvore degenerada; “média esperada” não é garantia diante de uma ordem adversarial. Na AVL, rotações e busca são garantidas por altura logarítmica. A construção completa da ABB com inserções em ordem pode levar Θ(n²); construir a AVL por n inserções custa O(n log n).

### 3.5 Execuções que falharam

As doze combinações (seis arquivos, duas fases), com cinco execuções por combinação, terminaram. Nenhuma execução oficial falhou ou deixou de terminar. As saídas do arquivo curto coincidiram linha a linha com os dois gabaritos oficiais recebidos.

## 4. Discussão

Se a chave fosse uma cadeia textual como `19_10_66600`, R3 e R5 dependeriam de comparação lexicográfica, que não necessariamente coincide com a ordem temporal. Uma representação que começa por dia e mês, omite o ano ou não preenche campos com zeros pode ordenar datas incorretamente, além de fazer anos diferentes colidirem. Para R3, a busca do menor texto maior ou igual ao limite poderia devolver uma chegada posterior/ anterior à cronológica correta. Para R5, o percurso por um intervalo textual poderia incluir ou excluir registros que não pertencem ao intervalo temporal. Uma chave textual só seria segura com componentes completos, ordem ano-mês-dia-hora em formato fixo e desempate de linha; a chave numérica composta adotada torna explícita a relação cronológica.

Um vetor ordenado poderia servir em vez de árvore se a carga fosse predominantemente estática e consultas dominassem: busca binária custaria O(log n), mas inserir/remover no meio de vetor exige deslocar Θ(n) elementos. Além disso, a ordenação inicial custa O(n log n), e a proibição do trabalho impede usar as rotinas prontas de ordenação no núcleo. Como o sistema recebe registros continuamente, as árvores balanceadas evitam os deslocamentos lineares e suportam atualizações em O(log n). A ABB foi mantida na Fase 1 como objeto de comparação didática, não como garantia de desempenho.

Os avisos são relatos de usuários e podem conter omissões, atrasos no envio, horários incorretos ou viés de amostragem. A ausência de registro não prova que o ônibus não passou. Para uma reclamação formal, seriam necessários volume e cobertura de observações, validação de horário e local, identificação de método de coleta, comparação com dados oficiais, tratamento de eventos excepcionais e documentação auditável. Os resultados descrevem as chegadas informadas, não provam causa de atrasos nem garantem o comportamento futuro do serviço.

**Comportamento mais custoso observado:** a inserção quase ordenada degenerou a ABB (altura 17.750 para N=20.000 e tempo de 10,017 s); a comparação com altura AVL 15 identificou o problema. Não foi observado erro de corretude nas cargas oficiais. O grupo informou que ainda não possui commits, e os registros semanais do redator não foram fornecidos; por isso, as decisões não podem ser atribuídas a hashes ou sessões específicas. As duas versões passaram também pelo arquivo pequeno oficial, que inclui remoções, empate no instante, consultas sem resultado e consultas que atravessam dias.

## 5. Conclusão

A solução separa a finalidade de cada fase. A ABB simples é adequada para demonstrar ordenação e operações de árvore, mas não garante que continue rápida com registros quase ordenados: a altura pode chegar a n e as buscas/atualizações degradam para Θ(n). A AVL mantém altura O(log n), e a altura pode ser obtida em O(1) quando armazenada nos nós. Para consultas específicas por linha, o índice secundário reduz PROXL a O(log ℓ + log n_l) e SEMANA a O(k(log ℓ + log n_l)), com espaço O(n + ℓ) adicional e custo de sincronização a cada inserção/remoção.

A hipótese de que a AVL evita degeneração ficou sustentada tanto pela teoria quanto pelas cargas oficiais: em quase ordem, sua altura final foi 11, 13 e 15, contra 889, 4.458 e 17.750 na ABB. Para N=20.000 quase ordenado, a mediana foi 232,539 ms na AVL e 10.017,246 ms na ABB; em dados embaralhados, porém, a ABB foi mais rápida até N=20.000 (146,683 ms contra 212,057 ms), ilustrando o custo de rotações e índices quando a ABB aleatória permanece rasa. A AVL usou mais memória rastreada (4,468 MiB contra 2,799 MiB no caso quase ordenado N=20.000), mas ofereceu altura limitada e consultas PROXL/SEMANA mais estáveis. As micro-medições mostraram que PROXL na ABB piora conforme o histórico cresce quando a linha não existe, enquanto a busca indexada ficou próxima de 0,00013 ms por consulta nesta execução. Os resultados quantitativos foram obtidos no sandbox cloud informado na Seção 2.1; o grupo pode repeti-los em seu próprio computador se necessário. A ausência de commits e de registros semanais limita a rastreabilidade histórica do desenvolvimento e deve ser considerada na avaliação do processo do grupo.

## Referências

CORMEN, Thomas H. et al. *Introduction to Algorithms*. 3. ed. Cambridge: MIT Press, 2009. Capítulos sobre árvores de busca e árvores balanceadas.

GOODRICH, Michael T.; TAMASSIA, Roberto; GOLDWASSER, Michael H. *Data Structures and Algorithms in Python*. 1. ed. Hoboken: Wiley, 2013. Seções sobre árvores de busca e árvores AVL.
