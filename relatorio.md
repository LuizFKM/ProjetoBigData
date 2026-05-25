# Relatório de Análise Exploratória

## Internações Respiratórias, Temperatura e PM2.5 em Palmas

**Projeto:** análise em Python/Jupyter das relações entre internações
respiratórias, condições térmicas e material particulado fino.
**Notebook analisado:** [cruzamentoDeDados.ipynb](cruzamentoDeDados.ipynb)
**Data do relatório:** 25 de maio de 2026.

## Resumo Executivo

Este projeto investiga se as internações respiratórias registradas em Palmas
apresentam associação temporal com a temperatura média e a concentração de
PM2.5. O notebook reúne três bases CSV em uma série diária, cria recortes
sazonais e mensais, calcula correlações, testa atrasos de 0 a 30 dias e
identifica dias com número atípico de internações.

Os principais achados exploratórios são:

- As internações exibem padrão sazonal: no período comparável entre as três
  bases, a média diária foi maior no inverno (`2,306`) e menor no verão
  (`1,374`).
- A associação entre temperatura e internações é negativa, porém fraca no
  nível diário (`r = -0,134` no período comparável) e mais visível na
  agregação mensal (`r = -0,425`).
- A análise de atrasos também encontrou correlações negativas modestas para
  temperatura, com o menor valor em aproximadamente 13 dias de atraso
  (`r = -0,166` no período comparável).
- O PM2.5 não apresentou correlação positiva com internações no tratamento
  utilizado: a correlação diária foi próxima de zero (`r = -0,017`) e nenhum
  lag entre 0 e 30 dias produziu correlação positiva.
- O ano de 2009 apresentou `1.078` internações, o maior total anual da série,
  seguido por 2010 (`995`). Esse período coincide com a pandemia de Influenza
  A/H1N1, devendo ser interpretado como potencialmente afetado por um fator
  epidemiológico externo.
- A série também atravessa a pandemia de COVID-19. Em comparação a 2019
  (`679` internações), os totais registrados foram menores em 2020 (`295`) e
  2021 (`397`), diferença que pode refletir alterações epidemiológicas,
  assistenciais ou de registro e não deve ser atribuída somente ao clima.

Os resultados descrevem associações e padrões temporais, mas **não demonstram
causalidade**. Para uma conclusão epidemiológica, seriam necessários controle
de confundidores, confirmação da procedência e unidades das bases, definição
rigorosa da janela de observação e modelos apropriados para contagens e séries
temporais.

## 1. Objetivo e Perguntas de Análise

O objetivo geral é avaliar como a variação temporal de temperatura e PM2.5 se
relaciona com o número de internações respiratórias em Palmas.

As perguntas tratadas pelo notebook são:

1. Existe correlação diária entre internações, temperatura e PM2.5?
2. A agregação por mês ou por estação torna um padrão sazonal mais evidente?
3. A exposição a temperaturas mais baixas ou à poluição pode anteceder
   internações em alguns dias?
4. Dias com internações excepcionalmente elevadas alteram as correlações?
5. Os anos de 2008, 2009 e 2010 têm comportamento especial que exige cautela
   interpretativa?

## 2. Estrutura do Projeto

| Arquivo | Função no projeto |
| --- | --- |
| [cruzamentoDeDados.ipynb](cruzamentoDeDados.ipynb) | Notebook de preparação, visualização e análise |
| [csv/internamentos_respiratorios_palmas.csv](csv/internamentos_respiratorios_palmas.csv) | Registros individuais de internações |
| [csv/particulas_ar_palmas.csv](csv/particulas_ar_palmas.csv) | Observações de PM2.5 |
| [csv/dados_climaticos.csv](csv/dados_climaticos.csv) | Observações de temperatura |
| [pyproject.toml](pyproject.toml) | Dependências e versão esperada de Python |
| [uv.lock](uv.lock) | Ambiente reprodutível resolvido pelo `uv` |

O projeto utiliza `pandas` para tratamento tabular, `matplotlib` e `seaborn`
para visualização, e Jupyter para execução interativa.

## 3. Bases de Dados

### 3.1 Variáveis utilizadas

| Base | Registros originais | Variáveis empregadas | Uso na análise |
| --- | ---: | --- | --- |
| Internações | 12.017 | `DT_INTER` | Contagem de internações por dia |
| Partículas do ar | 27.884 | `time`, `pm2p5` | Média diária de PM2.5 |
| Dados climáticos | 28.268 | `time`, `t2m` | Média diária de temperatura |

### 3.2 Cobertura temporal

| Série | Primeiro dia | Último dia | Dias diários disponíveis |
| --- | --- | --- | ---: |
| PM2.5 | 01/08/2006 | 31/08/2025 | 6.971 |
| Temperatura | 01/01/2007 | 07/05/2026 | 7.067 |
| Internações | 05/09/2007 | 25/12/2024 | 4.898 dias com ao menos uma internação |

As três fontes não começam nem terminam no mesmo momento. O intervalo em que
há cobertura ambiental e que está dentro da janela de internações vai de
**05/09/2007 a 25/12/2024**, totalizando **6.322 dias**.

### 3.3 Informações clínicas disponíveis, mas ainda não modeladas

A base de internações também contém `IDADE`, `DIAG_PRINC`, `DIAG_SECUN`,
`CID_NOTIF`, `CID_ASSO` e `CID_MORTE`. A análise atual utiliza somente a data
de internação. Uma leitura inicial indica:

| Faixa etária | Registros |
| --- | ---: |
| 0 a 4 anos | 4.351 |
| 5 a 14 anos | 2.849 |
| 15 a 59 anos | 2.135 |
| 60 anos ou mais | 2.682 |

Os códigos principais mais frequentes incluem `J180` (`3.266` registros),
`J159` (`1.577`) e `J189` (`1.408`). A interpretação clínica desses códigos
deve ser feita com dicionário CID validado e não é necessária para as
correlações temporais apresentadas neste notebook.

## 4. Preparação dos Dados

A função `preparar_dados()` incluída no notebook implementa o seguinte fluxo:

1. Lê os três arquivos CSV, sem alterar os dados originais.
2. Converte `DT_INTER` e `time` em datas diárias normalizadas.
3. Conta internações por dia com `groupby().size()`.
4. Calcula a média diária de `pm2p5`.
5. Calcula a média diária de `t2m`, renomeada para `temperatura`.
6. Padroniza a coluna temporal com o nome `data`.
7. Cria um calendário diário completo entre a menor e a maior data observada
   em qualquer fonte.
8. Faz os merges das três séries sobre esse calendário.
9. Preenche com zero os dias sem registro de internação.
10. Cria `pm25_ugm3 = pm2p5 * 1e9`, para tornar os números legíveis nos
    gráficos; multiplicar uma variável por uma constante positiva não altera
    sua correlação.
11. Acrescenta `ano`, `mes`, `ano_mes` e `estacao`.

As estações são definidas para o hemisfério sul:

| Estação | Meses |
| --- | --- |
| Verão | Dezembro, janeiro e fevereiro |
| Outono | Março, abril e maio |
| Inverno | Junho, julho e agosto |
| Primavera | Setembro, outubro e novembro |

## 5. Janela Temporal e Cuidado Metodológico

O notebook, conforme sua implementação, cria uma série de **7.220 dias** entre
01/08/2006 e 07/05/2026. Nessa série:

| Indicador | Valor |
| --- | ---: |
| Dias do calendário completo | 7.220 |
| Dias com temperatura ausente | 153 |
| Dias com PM2.5 ausente | 249 |
| Dias com internações igual a zero | 2.322 |
| Total de internações | 12.017 |

Há um ponto crucial: internações só estão cobertas na base entre 05/09/2007 e
25/12/2024. Portanto, preencher como zero datas anteriores ou posteriores a
essa cobertura pode significar **ausência de dados**, e não necessariamente
ausência de internações.

Por esse motivo, este relatório distingue:

| Cenário | Período | Uso |
| --- | --- | --- |
| Série implementada no notebook | 01/08/2006 a 07/05/2026 | Reproduz exatamente gráficos e resultados atuais |
| Período comparável recomendado | 05/09/2007 a 25/12/2024 | Testa robustez sem zerar internações fora da cobertura conhecida |

Dentro do período comparável, todos os 6.322 dias têm medição de temperatura e
PM2.5. Os dias sem internação nesse intervalo podem ser representados por zero
de forma mais defensável, desde que a base hospitalar seja completa para o
período.

## 6. Estatísticas Descritivas

### 6.1 Série completa implementada no notebook

| Variável | Observações válidas | Média | Mediana | Desvio padrão | Mínimo | Máximo |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Internações diárias | 7.220 | 1,664 | 1,000 | 1,789 | 0,000 | 22,000 |
| Temperatura | 7.067 | 16,745 | 17,514 | 3,998 | -0,081 | 25,300 |
| PM2.5 na escala de gráfico | 6.971 | 7,856 | 6,152 | 7,299 | 0,442 | 171,257 |

### 6.2 Resumo sazonal da série do notebook

| Estação | Dias | Internações: média | Internações: mediana | Internações: DP | Temperatura: média | PM2.5: média |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Verão | 1.805 | 1,188 | 1,000 | 1,369 | 20,024 | 5,965 |
| Outono | 1.816 | 1,846 | 1,000 | 1,826 | 16,692 | 6,031 |
| Inverno | 1.779 | 2,028 | 2,000 | 2,053 | 12,921 | 9,004 |
| Primavera | 1.820 | 1,601 | 1,000 | 1,733 | 17,302 | 10,397 |

O inverno apresenta a maior média diária de internações e a menor temperatura
média. O verão apresenta a menor média de internações e a maior temperatura
média. PM2.5 é mais alto, em média, na primavera e no inverno, mas esse
comportamento não se converte automaticamente em correlação positiva com
internações.

### 6.3 Sensibilidade no período comparável

| Estação | Média diária de internações |
| --- | ---: |
| Verão | 1,374 |
| Outono | 2,144 |
| Inverno | 2,306 |
| Primavera | 1,783 |

Ao retirar as datas fora da cobertura hospitalar, a ordem sazonal é preservada
e a diferença inverno-verão aumenta. Isso fortalece a indicação descritiva de
sazonalidade nas internações.

### 6.4 Padrão por mês do ano em meses completos do período comparável

Para evitar os meses parciais das extremidades, foi examinada a janela de
outubro de 2007 a novembro de 2024.

| Mês | Média diária de internações | Temperatura média | PM2.5 médio |
| ---: | ---: | ---: | ---: |
| Janeiro | 1,368 | 20,110 | 5,809 |
| Fevereiro | 1,435 | 20,037 | 5,885 |
| Março | 1,806 | 19,039 | 5,391 |
| Abril | 2,086 | 16,850 | 6,134 |
| Maio | 2,537 | 13,849 | 6,563 |
| Junho | 2,718 | 12,413 | 6,576 |
| Julho | 2,281 | 12,554 | 7,507 |
| Agosto | 1,934 | 14,032 | 12,215 |
| Setembro | 1,843 | 16,063 | 14,570 |
| Outubro | 1,837 | 17,333 | 9,488 |
| Novembro | 1,744 | 18,580 | 6,943 |
| Dezembro | 1,362 | 19,883 | 6,176 |

O maior valor médio de internações ocorre em junho (`2,718` por dia), quando a
temperatura média também está entre as mais baixas. O PM2.5 médio atinge maior
valor em setembro, sem coincidir com o pico de internações.

## 7. Correlações

O notebook utiliza correlação de Pearson, que mede associação linear. Valores
próximos de zero indicam ausência de associação linear forte, não ausência de
qualquer relação possível.

### 7.1 Correlação diária na série do notebook

| Variáveis | Correlação |
| --- | ---: |
| Internações e temperatura | -0,121 |
| Internações e PM2.5 | -0,019 |
| Temperatura e PM2.5 | 0,152 |

A relação diária entre internações e temperatura é negativa, mas fraca. A
relação diária entre internações e PM2.5 é praticamente nula.

### 7.2 Correlação mensal na série do notebook

| Variáveis | Correlação |
| --- | ---: |
| Internações e temperatura média | -0,334 |
| Internações e PM2.5 médio | -0,136 |
| Temperatura média e PM2.5 médio | -0,132 |

Após agregação mensal, a correlação negativa com temperatura aumenta em
magnitude. Esse resultado é compatível com um componente sazonal: meses mais
frios tendem a ter mais internações.

### 7.3 Teste de sensibilidade no período comparável

| Nível de análise | Internações x temperatura | Internações x PM2.5 |
| --- | ---: | ---: |
| Diário | -0,134 | -0,017 |
| Mensal | -0,425 | -0,157 |
| Mensal, somente meses completos | -0,425 | -0,137 |

O padrão principal é robusto à restrição temporal: a temperatura mantém
correlação negativa mais clara na escala mensal, enquanto PM2.5 permanece sem
relação positiva.

### 7.4 Correlação por estação na série do notebook

| Estação | Internações x temperatura | Internações x PM2.5 |
| --- | ---: | ---: |
| Verão | 0,024 | -0,006 |
| Outono | -0,121 | 0,007 |
| Inverno | 0,034 | -0,060 |
| Primavera | -0,003 | -0,037 |

Dentro de cada estação, as correlações são muito pequenas. Isto sugere que
parte importante da correlação mensal negativa com temperatura ocorre pela
diferença **entre estações**, não por uma variação linear forte dentro de cada
estação.

## 8. Médias Móveis

O notebook calcula médias móveis de 7 dias para:

- `total_internamentos`;
- `temperatura`;
- `pm25_ugm3`.

O objetivo é reduzir ruído diário e tornar visualmente perceptíveis tendências
de curto prazo. As médias móveis não substituem um modelo estatístico e não
estabelecem que uma curva causa variação em outra; elas servem como ferramenta
descritiva para localizar períodos de elevação ou queda simultânea.

## 9. Análise de Atraso Temporal

O teste de lags interpreta cada correlação como:

> internações de hoje comparadas com temperatura ou PM2.5 observados `X` dias
> antes.

Foram avaliados lags de 0 a 30 dias.

### 9.1 Temperatura

| Lag, em dias | Correlação na série do notebook | Correlação no período comparável |
| ---: | ---: | ---: |
| 13 | -0,152 | -0,166 |
| 12 | -0,150 | -0,165 |
| 7 | -0,150 | -0,166 |
| 4 | -0,150 | -0,166 |
| 6 | -0,150 | -0,166 |

As correlações negativas ficam um pouco mais fortes em atrasos próximos de 4 a
13 dias, mas continuam modestas. O resultado é compatível com a hipótese de
que períodos frios possam anteceder parte das internações, porém não permite
concluir que o frio seja sua causa.

### 9.2 PM2.5

| Lag, em dias | Maiores correlações observadas na série do notebook | Correlação no período comparável |
| ---: | ---: | ---: |
| 0 | -0,019 | -0,017 |
| 19 | -0,034 | -0,037 |
| 12 | -0,034 | -0,036 |
| 6 | -0,037 | -0,039 |
| 7 | -0,039 | -0,040 |

Nenhum lag apresentou correlação positiva entre PM2.5 e internações. Na série
do notebook, os valores variaram de `-0,068` a `-0,019`; no período comparável,
de `-0,075` a `-0,017`. Assim, este tratamento inicial não fornece evidência
de associação positiva linear defasada com PM2.5.

## 10. Possíveis Outliers

O notebook marca possíveis outliers pelo limite superior de Tukey:

`Q3 + 1,5 * IQR`.

Na série completa implementada:

| Medida | Valor |
| --- | ---: |
| Primeiro quartil (`Q1`) | 0 |
| Terceiro quartil (`Q3`) | 3 |
| IQR | 3 |
| Limite superior | 7,5 internações/dia |
| Dias sinalizados | 59 |

Os dez dias com mais internações foram:

| Data | Internações | Temperatura | PM2.5 |
| --- | ---: | ---: | ---: |
| 12/10/2009 | 22 | 15,298 | 22,034 |
| 29/06/2009 | 20 | 14,458 | 4,429 |
| 06/11/2009 | 20 | 20,919 | 9,670 |
| 27/02/2009 | 15 | 19,119 | 3,231 |
| 29/05/2009 | 14 | 12,809 | 9,272 |
| 30/03/2009 | 13 | 19,278 | 7,705 |
| 03/07/2012 | 12 | 15,610 | 2,779 |
| 05/07/2018 | 12 | 16,131 | 6,228 |
| 03/10/2008 | 11 | 15,766 | 2,560 |
| 11/05/2009 | 11 | 16,261 | 8,090 |

Seis dos dez maiores dias estão em 2009. Isso reforça a necessidade de
considerar eventos epidêmicos ou mudanças de atendimento/registro antes de
atribuir picos à temperatura ou à poluição.

### 10.1 Impacto dos outliers na correlação

| Relação | Com outliers | Sem outliers |
| --- | ---: | ---: |
| Internações x temperatura | -0,121 | -0,118 |
| Internações x PM2.5 | -0,019 | -0,021 |

A remoção apenas para comparação praticamente não modifica as correlações na
série do notebook. Logo, a conclusão de associação diária fraca não é
explicada exclusivamente pelos dias extremos.

No período comparável, o critério IQR sinaliza `111` dias, porque os zeros
artificiais fora da janela hospitalar deixam de influenciar a distribuição.
Essa diferença demonstra por que a definição da janela analítica precisa ser
formalizada antes de qualquer publicação ou inferência.

## 11. Análise de 2008, 2009 e 2010

| Ano | Total de internações | Mês com maior total | Internações no mês de pico | Temperatura média do pico | PM2.5 médio do pico |
| ---: | ---: | --- | ---: | ---: | ---: |
| 2008 | 660 | Outubro | 98 | 16,835 | 8,571 |
| 2009 | 1.078 | Agosto | 130 | 13,861 | 11,978 |
| 2010 | 995 | Junho | 131 | 12,474 | 6,954 |

Os maiores totais anuais de toda a série são:

| Posição | Ano | Internações |
| ---: | ---: | ---: |
| 1 | 2009 | 1.078 |
| 2 | 2010 | 995 |
| 3 | 2013 | 914 |
| 4 | 2017 | 903 |
| 5 | 2018 | 890 |

A Organização Mundial da Saúde declarou a pandemia de Influenza A/H1N1 em
**11 de junho de 2009**. A coincidência temporal não prova que os picos de
internação da base de Palmas decorreram de H1N1, pois este notebook não contém
confirmação laboratorial nem uma variável de circulação viral. Entretanto,
H1N1 é um confundidor epidemiológico plausível e deve ser considerado em
qualquer interpretação de 2009 e do período subsequente.

### 11.1 Período da COVID-19

A base de internações continua até 25/12/2024 e, portanto, também atravessa a
pandemia de COVID-19. O primeiro caso confirmado no Brasil foi registrado em
**26 de fevereiro de 2020**, e a Organização Mundial da Saúde caracterizou a
COVID-19 como pandemia em **11 de março de 2020**.

| Ano | Internações registradas | Variação em relação a 2019 |
| ---: | ---: | ---: |
| 2019 | 679 | Referência |
| 2020 | 295 | -56,6% |
| 2021 | 397 | -41,5% |
| 2022 | 515 | -24,2% |
| 2023 | 473 | -30,3% |
| 2024 | 374 | -44,9% |

Ao contrário do observado para 2009, o período inicial da COVID-19 não aparece
na base como aumento dos totais anuais; há redução acentuada em 2020 e 2021.
Essa queda não permite concluir que a pandemia tenha reduzido doenças
respiratórias em Palmas. Entre explicações possíveis estão:

- redução de circulação de algumas infecções respiratórias devido a isolamento
  e uso de máscaras;
- mudança na procura por atendimento hospitalar durante a pandemia;
- alteração na disponibilidade de leitos ou nos critérios de internação;
- registro de hospitalizações por COVID-19 em códigos ou bases diferentes dos
  diagnósticos respiratórios selecionados neste arquivo;
- mudanças de completude ou extração dos registros.

Assim, 2020 a 2024 também devem ser tratados como período potencialmente
afetado por um evento externo. Uma análise confirmatória deveria identificar
quais diagnósticos entram na extração, verificar se internações por COVID-19
estão incluídas e testar resultados com um indicador específico para a
pandemia.

## 12. Gráficos Disponíveis no Notebook

O notebook contém visualizações para apoiar a inspeção dos resultados:

| Visualização | Pergunta apoiada |
| --- | --- |
| Série temporal original de internações, PM2.5 e temperatura | Como as variáveis evoluem ao longo do tempo? |
| Barras de internações médias por estação | Existe sazonalidade nas internações? |
| Barras de temperatura média por estação | As estações foram representadas coerentemente? |
| Total mensal de internações | Quais períodos concentram picos? |
| Internações mensais e temperatura média | Picos coincidem visualmente com meses frios? |
| Médias móveis de 7 dias | Há tendências temporais de curto prazo? |
| Heatmaps diário e mensal | Qual a força da correlação linear geral? |
| Heatmaps por estação | A relação persiste dentro das estações? |
| Linha de correlação por lag | Exposições anteriores têm associação maior? |
| Destaque de outliers | Quais dias têm contagens atípicas? |

PM2.5 aparece nas séries temporais, nos heatmaps e na análise de lags. Uma
visualização complementar útil seria um gráfico de dispersão entre
`pm25_ugm3` e `total_internamentos`, com transparência para os pontos e linha
de tendência, deixando visualmente evidente a associação linear fraca.

## 13. Interpretação Integrada

### Temperatura

Os dados mostram uma associação negativa consistente em nível agregado:
estações e meses mais frios apresentam maior número médio de internações. A
associação é pequena no nível diário e quase desaparece quando se observa a
variação dentro de cada estação, o que indica forte componente sazonal.

### PM2.5

O PM2.5 não apresentou correlação positiva forte, nem no nível diário, nem
mensal, nem com atraso de até 30 dias. Isso não demonstra que poluição não
afete doenças respiratórias; significa apenas que esta base, esta escala
temporal e este método exploratório não detectaram a relação esperada.

### H1N1, COVID-19 e eventos externos

O grande número de internações em 2009 e 2010, juntamente com a concentração
de dias extremos em 2009, impede interpretar todos os picos como consequência
de temperatura ou PM2.5. Um indicador temporal para pandemia, ou dados de
circulação viral, seriam necessários para separar esses efeitos.

Da mesma forma, os totais menores observados a partir de 2020 não podem ser
interpretados diretamente como efeito climático: a pandemia de COVID-19 pode
ter alterado a ocorrência de síndromes respiratórias, a procura por serviços,
a codificação dos casos e a própria cobertura da base.

## 14. Limitações

1. **Janela de internações:** a série atual atribui zero a dias fora da
   cobertura conhecida da base hospitalar. A análise principal deveria ser
   repetida na janela comum ou com uma coluna explícita de cobertura.
2. **Procedência e unidade:** o repositório contém os CSVs, mas não documenta
   sua fonte oficial, processo de extração ou unidade original de `pm2p5`.
   Isso precisa ser confirmado antes de apresentar `pm25_ugm3` como unidade
   física validada.
3. **Correlação não é causalidade:** temperatura e poluição podem estar
   associadas a estação, umidade, circulação viral, queimadas, feriados,
   população, acesso à saúde e mudanças nos registros.
4. **Modelo para contagens:** internações são contagens diárias, com muitos
   zeros e picos. Correlação de Pearson é apenas uma primeira aproximação;
   modelos de Poisson ou binomial negativa seriam mais apropriados.
5. **Lags múltiplos:** testar diversos atrasos aumenta a chance de destacar
   associações por acaso; resultados deveriam ser validados com modelo
   previamente especificado.
6. **Ausência de estratificação:** crianças e idosos podem reagir de modo
   diferente às exposições; o notebook ainda não usa idade ou diagnóstico.
7. **H1N1 e COVID-19:** a base não identifica diretamente casos pandêmicos; a
   influência desses eventos é uma hipótese contextual plausível, não uma
   classificação dos registros.

## 15. Recomendações para Continuidade

1. Restringir a análise principal ao período comum de 05/09/2007 a
   25/12/2024, ou registrar explicitamente a cobertura de cada fonte.
2. Confirmar a origem, a unidade e a qualidade dos dados de PM2.5 e
   temperatura.
3. Acrescentar gráficos de dispersão para internações versus PM2.5 e
   internações versus temperatura, tanto diários quanto mensais.
4. Analisar internações por faixas etárias e grupos diagnósticos.
5. Testar modelos de regressão para contagens, incorporando sazonalidade,
   tendência de longo prazo e lags distribuídos.
6. Incluir indicadores para os períodos de H1N1 e COVID-19, ou séries externas
   de circulação viral e hospitalizações confirmadas.
7. Avaliar outras variáveis ambientais potencialmente relevantes, como
   umidade, precipitação e queimadas, caso estejam disponíveis.

## 16. Reprodutibilidade

O ambiente está definido em [pyproject.toml](pyproject.toml) e
[uv.lock](uv.lock). Para executar o notebook:

```powershell
uv sync
uv run jupyter lab
```

Em seguida, deve-se abrir [cruzamentoDeDados.ipynb](cruzamentoDeDados.ipynb) e
executar as células em ordem. Os CSVs originais permanecem inalterados; todas
as transformações são realizadas em memória.

## Referências

- Dados locais utilizados na análise: arquivos CSV presentes no diretório
  [`csv/`](csv/).
- World Health Organization. *DG Statement Following the Meeting of the
  Emergency Committee*, 11 de junho de 2009. Disponível em:
  <https://www.who.int/news/item/11-06-2009-dg-statement-following-the-meeting-of-the-emergency-committee>.
- Ministério da Saúde. *Brasil confirma primeiro caso da doença*, 26 de
  fevereiro de 2020. Disponível em:
  <https://www.gov.br/saude/pt-br/assuntos/noticias/2020/fevereiro/brasil-confirma-primeiro-caso-de-novo-coronavirus>.
- World Health Organization. *WHO Timeline - COVID-19*, com registro de 11 de
  março de 2020. Disponível em:
  <https://www.who.int/news/item/27-04-2020-who-timeline---covid-19>.
