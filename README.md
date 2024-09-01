# Extraindo e Visualizando Dados de uma API Paginada no Power BI

## Introdução
Este tutorial demonstra como extrair, modelar e visualizar dados de uma API paginada (ex: Rick and Morty) no Power BI. Utilizaremos o Power Query para criar uma função personalizada que busca os dados de cada página da API e, em seguida, combinamos os resultados em uma única tabela.

## Pré-requisitos
* Power BI Desktop
* Conhecimento básico da linguagem M do Power Query

## Passo a Passo

1. **[Conectar à API:](https://rickandmortyapi.com/documentation)**
   * Crie uma nova consulta no Power Query.
   * Utilize a função `Web.Contents` para fazer uma requisição à API, utilizando o parâmetro `RelativePath` para construir a URL dinamicamente.
   * Defina uma função personalizada que recebe o número da página como parâmetro.
```
(page as text)=>

let
    Fonte = Json.Document(
        Web.Contents(
            "https://rickandmortyapi.com/",
            [RelativePath = "api/character?page="&Text.From(page)]      ) 
    ),

    #"Convertido para Tabela" = Table.FromRecords({Fonte}),
    #"info Expandido" = Table.ExpandRecordColumn(#"Convertido para Tabela", "info", {"count", "pages", "next", "prev"}, {"info.count", "info.pages", "info.next", "info.prev"}),
    #"results Expandido" = Table.ExpandListColumn(#"info Expandido", "results"),
    #"results Expandido1" = Table.ExpandRecordColumn(#"results Expandido", "results", {"id", "name", "status", "species", "type", "gender", "origin", "location", "image", "episode", "url", "created"}, {"results.id", "results.name", "results.status", "results.species", "results.type", "results.gender", "results.origin", "results.location", "results.image", "results.episode", "results.url", "results.created"}),
    #"results.origin Expandido" = Table.ExpandRecordColumn(#"results Expandido1", "results.origin", {"name", "url"}, {"results.origin.name", "results.origin.url"}),
    #"results.location Expandido" = Table.ExpandRecordColumn(#"results.origin Expandido", "results.location", {"name", "url"}, {"results.location.name", "results.location.url"}),
    #"results.episode Expandido" = Table.ExpandListColumn(#"results.location Expandido", "results.episode"),
    #"Tipo Alterado" = Table.TransformColumnTypes(#"results.episode Expandido",{{"info.count", Int64.Type}, {"info.pages", Int64.Type}, {"info.next", type text}, {"info.prev", type any}, {"results.id", Int64.Type}, {"results.name", type text}, {"results.status", type text}, {"results.species", type text}, {"results.type", type any}, {"results.gender", type text}, {"results.origin.name", type text}, {"results.origin.url", type text}, {"results.location.name", type text}, {"results.location.url", type text}, {"results.image", type text}, {"results.episode", type text}, {"results.url", type text}, {"results.created", type datetime}})
in
    #"Tipo Alterado"
```
2. **Usar a função para retornar o número de páginas:**
   * Insira um número qualquer de página na função para retornar uma tabela.
   * Selecione a coluna com o número total de páginas e exclua todas as outras.
   * Remova as duplicatas, dessa forma restará apenas uma linha na coluna.
   * Criei uma nova coluna utilizando a função `List.Numbers` para gerar uma lista com todos os números de páginas.
```
=List.Numbers(1, [nome_da_coluna_com_o_total_de_paginas])
```
   * Selecione a nova coluna e altere seu tipo de dados para texto.
   * Invoque a função criada e selecione a coluna com os números de páginas.
   * Após esse processo o Power Query irá extrair cada página da API seguinte a sequência numérica da coluna que foi criada.

4. **Modelar os dados:**
   * Expandir as colunas para obter os dados desejados.
   * Alterar os tipos de dados das colunas, se necessário.

5. **Criar visualizações:**
   * Carregar os dados no modelo de dados do Power BI.
   * Criar gráficos e tabelas para visualizar os dados.


[<img src="https://i.ibb.co/JqpWrfc/Rick-e-Morty.png" alt="capa" border="0">](https://x.gd/egkIS)

[Relatório de Personagens](https://x.gd/egkIS)
