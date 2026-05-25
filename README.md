# Internacoes Respiratorias, Clima e Poluicao em Palmas

Projeto de analise exploratoria em Python/Jupyter sobre a relacao entre
internacoes por doencas respiratorias, concentracao de particulas finas
(`PM2.5`) e temperatura do ar em Palmas.

O notebook principal combina as tres fontes em escala diaria e investiga se
padroes sazonais ou efeitos com atraso temporal ajudam a explicar relacoes que
nao aparecem com clareza em uma correlacao diaria simples.

## Arquivos

```text
.
|-- cruzamentoDeDados.ipynb
|-- csv/
|   |-- internamentos_respiratorios_palmas.csv
|   |-- particulas_ar_palmas.csv
|   `-- dados_climaticos.csv
|-- pyproject.toml
`-- uv.lock
```

## Bases Utilizadas

| Arquivo | Informacao utilizada | Colunas principais |
| --- | --- | --- |
| `internamentos_respiratorios_palmas.csv` | Registros de internacoes respiratorias | `DT_INTER` |
| `particulas_ar_palmas.csv` | Concentracao de particulas finas | `time`, `pm2p5` |
| `dados_climaticos.csv` | Temperatura do ar | `time`, `t2m` |

As bases tem coberturas temporais diferentes:

| Serie | Inicio | Fim |
| --- | --- | --- |
| PM2.5 | 01/08/2006 | 31/08/2025 |
| Temperatura | 01/01/2007 | 07/05/2026 |
| Internacoes | 05/09/2007 | 25/12/2024 |

Por isso, alguns dias no inicio ou no final da serie consolidada podem possuir
valores ausentes para variaveis que ainda nao estavam disponiveis, ou que
deixaram de ser medidas.

## Metodologia

O notebook [cruzamentoDeDados.ipynb](cruzamentoDeDados.ipynb) realiza:

1. Leitura e normalizacao diaria das datas dos tres CSVs.
2. Contagem diaria de internacoes e calculo das medias diarias de temperatura e PM2.5.
3. Criacao de um calendario diario completo, preenchendo dias sem internacao com zero.
4. Conversao de `pm2p5` para a coluna de leitura facilitada `pm25_ugm3`.
5. Classificacao por ano, mes e estacao do hemisferio sul.
6. Resumos estatisticos e graficos por estacao e por mes.
7. Medias moveis de 7 dias para identificar tendencias temporais.
8. Matrizes de correlacao diaria, mensal e separada por estacao.
9. Correlacoes com atrasos (`lags`) de 0 a 30 dias para temperatura e PM2.5.
10. Identificacao de possiveis outliers de internacoes pelo metodo IQR, sem remocao automatica.
11. Recorte dos anos de 2008, 2009 e 2010, considerando a possivel influencia externa da pandemia de Influenza A/H1N1.

## Como Executar

O projeto utiliza `uv` para gerenciar o ambiente e as dependencias.

Requisitos:

- Python 3.14 ou superior, conforme `pyproject.toml`;
- `uv` instalado.

No diretorio do projeto:

```powershell
uv sync
uv run jupyter lab
```

No Jupyter Lab, abra `cruzamentoDeDados.ipynb` e execute as celulas em ordem.
Tambem e possivel iniciar a interface classica:

```powershell
uv run jupyter notebook
```

As dependencias principais sao:

- `pandas`;
- `matplotlib`;
- `seaborn`;
- `jupyterlab` / `notebook`.

## Analises e Graficos

O notebook apresenta:

- serie temporal de internacoes, PM2.5 e temperatura;
- barras com media diaria de internacoes e temperatura media por estacao;
- total mensal de internacoes;
- comparacao mensal entre internacoes e temperatura;
- medias moveis de 7 dias;
- heatmaps de correlacao;
- curva de correlacao por lag temporal;
- visualizacao de dias sinalizados como possiveis outliers;
- tabela mensal para 2008, 2009 e 2010.

## Resultados Exploratorios

Na preparacao atual dos dados:

- a media diaria de internacoes e maior no inverno (`2,03`) e menor no verao (`1,19`);
- a correlacao entre internacoes e temperatura e fraca no nivel diario (`-0,121`), mas fica mais negativa no nivel mensal (`-0,334`);
- o PM2.5 nao apresentou correlacao positiva forte com internacoes, inclusive nos lags avaliados;
- 59 dias foram sinalizados como possiveis outliers pelo criterio IQR.

Esses resultados sugerem um padrao sazonal, mas nao demonstram causalidade.
Internacoes respiratorias tambem podem ser influenciadas por circulacao viral,
perfil populacional, mudancas de registro, outros poluentes e eventos
epidemiologicos.

## Observacoes Importantes

- Os arquivos CSV originais nao sao alterados durante a analise.
- Dias sem registros de internacao sao considerados como zero na serie diaria.
- Ausencias de temperatura ou PM2.5 sao preservadas quando nao existe medicao correspondente.
- Um outlier nao e tratado automaticamente como erro; pode representar um evento real.
- O periodo de 2009 e 2010 deve ser interpretado com cuidado devido a possivel influencia da pandemia de H1N1.
