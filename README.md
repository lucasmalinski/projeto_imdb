# Projeto IMDB: Dashboard de Performance Cinematográfica

> Análise exploratória e financeira de um dataset com mais de 33 mil filmes, focando em rentabilidade e correlação crítica.

> Nota ⚠️: Disponível apenas dentre minha organização. 
[Painel no Power BI Service](https://app.powerbi.com/links/wzaglYlxI3?ctid=dfb66dc4-3f3c-492c-991d-727dbd1c89d4&pbi_source=linkShare)

---

## Perguntas Iniciais de Negócio

> - **Rentabilidade por País:** Filmes de determinadas regiões são lucrativos?
> - **Qualidade vs. Prêmios:** Existe correlação entre a nota IMDB e o sucesso no Oscar?
> - **Receita:** Quais gêneros dominam o faturamento global?

---

## Processo de ETL - Etapas Aplicadas 

### Script M — Passo a Passo
 
**PASSO 0 — Leitura do CSV e tipagem inicial (todas as colunas como texto)**
 
```powerquery
Fonte = Csv.Document(
    File.Contents("...\filmes_nova - world_imdb_movies_top_movies_per_year.csv"),
    [Delimiter=",", Columns=24, Encoding=65001, QuoteStyle=QuoteStyle.None]
),
 
#"PASSO 0 [AUTO] - Tipo Alterado" = Table.TransformColumnTypes(Fonte,{
    {"Column1", type text}, {"Column2", type text}, {"Column3", type text},
    {"Column4", type text}, {"Column5", type text}, {"Column6", type text},
    {"Column7", type text}, {"Column8", type text}, {"Column9", type text},
    {"Column10", type text}, {"Column11", type text}, {"Column12", type text},
    {"Column13", type text}, {"Column14", type text}, {"Column15", type text},
    {"Column16", type text}, {"Column17", type text}, {"Column18", type text},
    {"Column19", type text}, {"Column20", type text}, {"Column21", type text},
    {"Column22", type text}, {"Column23", type text}, {"Column24", type text}
})
```
 
---
 
**PASSO 7 — Remoção das 2 primeiras linhas e promoção de cabeçalhos**
 
```powerquery
#"PASSO 7 - 2 Linhas Superiores Removidas" = Table.Skip(
    #"PASSO 0 [AUTO] - Tipo Alterado", 2
),
 
#"PASSO 7 - Cabeçalhos Promovidos" = Table.PromoteHeaders(
    #"PASSO 7 - 2 Linhas Superiores Removidas", [PromoteAllScalars=true]
),
 
#"PASSO 0 [AUTO] - Tipos Alterados - (automático)" = Table.TransformColumnTypes(
    #"PASSO 7 - Cabeçalhos Promovidos",{
        {"id", type text}, {"title", type text}, {"link", type text},
        {"year", Int64.Type}, {"duration", type text}, {"rating_mpa", type text},
        {"rating_imdb", Int64.Type}, {"vote", Int64.Type}, {"budget", Int64.Type},
        {"gross_world_wide", Int64.Type}, {"gross_us_canada", Currency.Type},
        {"gross_opening_weekend", Int64.Type}, {"director", type text},
        {"writer", type text}, {"star", type text}, {"genre", type text},
        {"country_origin", type text}, {"filming_location", type text},
        {"production_company", type text}, {"language", type text},
        {"captions", type text}, {"win", Int64.Type},
        {"nomination", Int64.Type}, {"oscar", Int64.Type}
    }
)
```
 
---
 
**PASSO 8 — Remoção de colunas desnecessárias (link, captions, win)**
 
```powerquery
#"PASSO 8 - Colunas Removidas (link)" = Table.RemoveColumns(
    #"PASSO 0 [AUTO] - Tipos Alterados - (automático)",
    {"link", "captions", "win"}
)
```
 
---
 
**PASSO 9 — Limpeza do ID (remove prefixo "tt") e remoção de duplicatas**
 
```powerquery
#"PASSO 9 - Últimos caracteres extraídos (remover leading tt)" = Table.TransformColumns(
    #"PASSO 8 - Colunas Removidas (link)",
    {{"id", each Text.End(_, 7), type text}}
),
 
#"PASSO 9 - Duplicatas Removidas (por id)" = Table.Distinct(
    #"PASSO 9 - Últimos caracteres extraídos (remover leading tt)", {"id"}
)
```
 
---
 
**PASSO 10 — Renomeação de todas as colunas para português**
 
```powerquery
#"PASSO 10 - Colunas Renomeadas (todas)" = Table.RenameColumns(
    #"PASSO 9 - Duplicatas Removidas (por id)",{
        {"title", "título"}, {"year", "ano"}, {"duration", "duração"},
        {"rating_mpa", "nota_mpa"}, {"rating_imdb", "nota_imdb"},
        {"vote", "voto"}, {"budget", "orçamento"},
        {"gross_world_wide", "receita_bruta_global"},
        {"gross_us_canada", "receita_bruta_us_canada"},
        {"director", "diretor"}, {"writer", "roteirista"},
        {"gross_opening_weekend", "bruto_lançamento"},
        {"star", "celebridade"}, {"genre", "gênero"},
        {"filming_location", "local_de_filmagem"},
        {"production_company", "produtora"},
        {"country_origin", "país de origem"},
        {"language", "idioma"}, {"nomination", "indicações"}
    }
)
```
 
---
 
**PASSO 11 — Tipagem monetária de orçamento e receita**
 
```powerquery
#"PASSO 11 - Tipo Alterado2 (orçamento)" = Table.TransformColumnTypes(
    #"PASSO 10 - Colunas Renomeadas (todas)",{
        {"receita_bruta_global", Currency.Type},
        {"orçamento", Currency.Type}
    }
)
```
 
---
 
**PASSO 12 — Padronização do título em maiúsculas**
 
```powerquery
#"PASSO 12 - Texto em Maiúscula (título)" = Table.TransformColumns(
    #"PASSO 11 - Tipo Alterado2 (orçamento)",
    {{"título", Text.Upper, type text}}
)
```
 
---
 
**PASSO 13 — Extração do gênero principal (antes da primeira vírgula)**
 
```powerquery
#"PASSO 13 - Dividir Coluna por Delimitador (gênero)" = Table.SplitColumn(
    #"PASSO 12 - Texto em Maiúscula (título)", "gênero",
    Splitter.SplitTextByEachDelimiter({","}, QuoteStyle.Csv, false),
    {"gênero.1", "gênero.2"}
),
 
#"PASSO 13 - Tipos Alterados (automático)" = Table.TransformColumnTypes(
    #"PASSO 13 - Dividir Coluna por Delimitador (gênero)",{
        {"gênero.1", type text}, {"gênero.2", type text}, {"id", Int64.Type}
    }
),
 
#"PASSO 13 - Colunas Removidas (gênero.2)" = Table.RemoveColumns(
    #"PASSO 13 - Tipos Alterados (automático)", {"gênero.2"}
),
 
#"PASSO 13 - Colunas Renomeadas (gênero.1 -> gênero)" = Table.RenameColumns(
    #"PASSO 13 - Colunas Removidas (gênero.2)", {{"gênero.1", "gênero"}}
)
```
 
---
 
**PASSO 14 — Remoção de linhas com nota_imdb nula ou vazia**
 
```powerquery
#"PASSO 14 - Linhas Filtradas1 (nota_imdb nulas)" = Table.SelectRows(
    #"PASSO 13 - Colunas Renomeadas (gênero.1 -> gênero)",
    each ([nota_imdb] <> null and [nota_imdb] <> "")
)
```
 
---
 
**PASSO 15 — Conversão de duração: texto (ex: "2h 15m") → total em minutos (Int64)**
 
```powerquery
#"PASSO 15 - Dividir Coluna de duração baseado em dígito/não dígito" = Table.SplitColumn(
    #"PASSO 14 - Linhas Filtradas1 (nota_imdb nulas)", "duração",
    Splitter.SplitTextByCharacterTransition(
        {"0".."9"}, (c) => not List.Contains({"0".."9"}, c)
    ),
    {"duração.1", "duração.2", "duração.3"}
),
 
#"PASSO 15 - Valor Substituído [1 (h) equivale a 60(min)]"  = Table.ReplaceValue(#"PASSO 15 - Dividir Coluna de duração baseado em dígito/não dígito","1","60",Replacer.ReplaceValue,{"duração.1"}),
#"PASSO 15 - Valor Substituído1 [2 (h) equivale a 120(min)]" = Table.ReplaceValue(#"PASSO 15 - Valor Substituído [1 (h) equivale a 60(min)]","2","120",Replacer.ReplaceValue,{"duração.1"}),
#"PASSO 15 - Valor Substituído2 [3 (h) equivale a 180(min)]" = Table.ReplaceValue(#"PASSO 15 - Valor Substituído1 [2 (h) equivale a 120(min)]","3","180",Replacer.ReplaceValue,{"duração.1"}),
#"PASSO 15 - Valor Substituído3 [4 (h) equivale a 240(min)]" = Table.ReplaceValue(#"PASSO 15 - Valor Substituído2 [3 (h) equivale a 180(min)]","4","240",Replacer.ReplaceValue,{"duração.1"}),
 
#"PASSO 15 - Dividir Coluna por Delimitador (Extrair ""m"" da coluna de minutos)" = Table.SplitColumn(
    #"PASSO 15 - Valor Substituído3 [4 (h) equivale a 240(min)]", "duração.2",
    Splitter.SplitTextByDelimiter(" ", QuoteStyle.Csv),
    {"duração.2.1", "duração.2.2"}
),
 
#"PASSO 15 - Colunas de duração como Int64" = Table.TransformColumnTypes(
    #"PASSO 15 - Dividir Coluna por Delimitador (Extrair ""m"" da coluna de minutos)",{
        {"duração.2.1", type text}, {"duração.2.2", Int64.Type},
        {"duração.3", type text}, {"duração.1", Int64.Type}
    }
),
 
#"PASSO 15 - Colunas Removidas (coluna com string 'm' e coluna com string 'h')" = Table.RemoveColumns(
    #"PASSO 15 - Colunas de duração como Int64", {"duração.2.1", "duração.3"}
),
 
#"PASSO 15 - Erros Substituídos (classificação indicativa em duração)" = Table.ReplaceErrorValues(
    #"PASSO 15 - Colunas Removidas (coluna com string 'm' e coluna com string 'h')",
    {{"duração.1", 0}}
),
 
#"PASSO 15 - Somar a coluna de hora convertida à de minutos" = Table.AddColumn(
    #"PASSO 15 - Erros Substituídos (classificação indicativa em duração)",
    "Personalizar", each [duração.1] + [duração.2.2]
),
 
#"PASSO 15 - Colunas Reordenadas" = Table.ReorderColumns(
    #"PASSO 15 - Somar a coluna de hora convertida à de minutos",{
        "id", "título", "ano", "duração.1", "duração.2.2", "Personalizar",
        "nota_mpa", "nota_imdb", "voto", "orçamento", "receita_bruta_global",
        "receita_bruta_us_canada", "bruto_lançamento", "diretor", "roteirista",
        "celebridade", "gênero", "país de origem", "local_de_filmagem",
        "produtora", "idioma", "indicações", "oscar"
    }
),
 
#"PASSO 15 - Colunas Renomeadas (DURAÇÃO)" = Table.RenameColumns(
    #"PASSO 15 - Colunas Reordenadas",{
        {"Personalizar", "duração"},
        {"duração.1", "duração.horas"},
        {"duração.2.2", "duração.minutos"}
    }
),
 
#"PASSO 15 - Colunas Removidas (duração.minutos e duração.horas)" = Table.RemoveColumns(
    #"PASSO 15 - Colunas Renomeadas (DURAÇÃO)", {"duração.horas", "duração.minutos"}
),
 
#"PASSO 15 - Tipo Alterado (duração -> int64)" = Table.TransformColumnTypes(
    #"PASSO 15 - Colunas Removidas (duração.minutos e duração.horas)",
    {{"duração", Int64.Type}}
)
```
 
---
 
**PASSO 16 — Tratamento de falsos nulos em local_de_filmagem**
 
```powerquery
#"PASSO 16 - Falsos nulos" = Table.ReplaceValue(
    #"PASSO 15 - Tipo Alterado (duração -> int64)",
    "", null, Replacer.ReplaceValue, {"local_de_filmagem"}
)
```
 
---
 
**PASSO 17 — Coluna condicional: classificação por duração (curta vs. longa metragem)**
 
```powerquery
#"PASSO 17 - Coluna condicional - classificacao_duracao" = Table.AddColumn(
    #"PASSO 16 - Falsos nulos", "classificacao_duracao",
    each if [duração] = null then null
         else if [duração] <= 65 then "curta_metragem"
         else if [duração] > 65  then "longa_metragem"
         else null
),
 
#"PASSO 17 - Reoordenar classificacao_duracao" = Table.ReorderColumns(
    #"PASSO 17 - Coluna condicional - classificacao_duracao",{
        "id", "título", "ano", "duração", "classificacao_duracao",
        "nota_mpa", "nota_imdb", "voto", "orçamento", "receita_bruta_global",
        "receita_bruta_us_canada", "bruto_lançamento", "diretor", "roteirista",
        "celebridade", "gênero", "país de origem", "local_de_filmagem",
        "produtora", "idioma", "indicações", "oscar"
    }
)
```
 
---
 
**PASSO 19 — Substituição de nulos em local_de_filmagem por "Desconhecido"**
 
```powerquery
#"PASSO 19 - Nova coluna condicional - local_de_filmagem2" = Table.AddColumn(
    #"PASSO 17 - Reoordenar classificacao_duracao", "Personalizar",
    each if [local_de_filmagem] <> null then [local_de_filmagem] else "Desconhecido"
),
 
#"PASSO 19 - Removida local_de_filmagem1" = Table.RemoveColumns(
    #"PASSO 19 - Nova coluna condicional - local_de_filmagem2", {"local_de_filmagem"}
),
 
#"PASSO 19 - Renomear [local_de_filmagem2 -> local_de_filmagem]" = Table.RenameColumns(
    #"PASSO 19 - Removida local_de_filmagem1", {{"Personalizar", "local_de_filmagem"}}
)
```
 
---

## Inteligência de Dados (DAX)

```dax
-- Resultado Financeiro Total
Lucro = SUM(receita_bruta_global) - SUM(orçamento)

-- Status de Crítica
status_de_critica = IF([nota_imdb] > 80, "Excelente", "Comum") 

-- Média Pós-2000
Media De Nota IMDB pós Anos 2000 = CALCULATE(
    AVERAGE(imdb_movies[nota_imdb]),
    imdb_movies[ano] >= 2000
    )

-- Resultado Financeiro Total 
Resultado Financeiro Total = 
SUMX(
    'imdb_movies', 
    'imdb_movies'[receita_bruta_global] - 'imdb_movies'[orçamento]
)
```

---

## Dashboard

*Visualizações de densidade, scatter plots e rankings de receita por gênero disponíveis no relatório Power BI.*
> Nota ⚠️: Disponível apenas dentre minha organização. 
[Painel no Power BI Service](https://app.powerbi.com/links/wzaglYlxI3?ctid=dfb66dc4-3f3c-492c-991d-727dbd1c89d4&pbi_source=linkShare)
