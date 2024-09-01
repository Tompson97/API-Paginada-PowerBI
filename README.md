# API-Paginada-PowerBI
Técnica de extração, paginação, modelagem e publicação de um relatório de personagens da animação Rick e Morty no Power BI.

[API Rick e Morty](https://rickandmortyapi.com/)

[<img src="https://i.ibb.co/JqpWrfc/Rick-e-Morty.png" alt="capa" border="0">](https://x.gd/egkIS)

[Relatório de Personagens](https://x.gd/egkIS)

Função para extrair os dados de cada página da API usando o editor Avançando do Power Query.\
Neste mesmo código usei o RelativePath para quebrar a URL de consulta, dessa forma as atualizações agendadas podem ocorrer no Power Bi Serviços.\
As linhas de códigos que iniciam com # são para tratamento de dados, inseridas automaticamente pelo Power Query, mas são opcionais.
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

Após inserir esse código no editor avançado do Power Query uma função será criada e solicitará o valor da variável "page" que determinará o número de página. Você pode testar inserindo o número da página desejada, mas precisamos extrair os dados de todas as páginas de uma vez para seguir com o tratamento, limpeza e posteriormente com a construção do relatório no Power BI.

Para descobrir o número total de páginas faça um teste na função inserindo a primeira página, a API irá retornar uma tabela com vários colunas e uma delas irá conter o total de páginas "pages". Remova todas as outras colunas e exclua as linhas duplicadas, isso irá resultar em uma tabela contendo apenas uma linha com uma coluna com o total de páginas existentes. \
O próximo passo será criar uma coluna com uma lista que vai de 1 até o numero total de páginas, acesse o meno "Adicionar Coluna" do Power Query e selecione "Coluna Personalizada" e insira o seguinte comando:\
```
=List.Numbers(1, [nome_da_coluna_com_o_total_de_paginas])
```
Pressione "OK" uma nova coluna irá surgir e você precisa expandir os dados para exibir todas as linhas da lista criada.\
Altere o tipo de dados da coluna com os números de paginas para texto.\
Com a nova coluna criada vá novamente ao menu "Adicionar Coluna" selecione "Invocar Função Personalizada" selecione a função que foi criada no início da nossa consulta e selecione a coluna onde estão as sequência de números de cada página.\

Pronto!
A extração dos dados de todas as páginas foi criado, você pode seguir com a limpeza e tratamento dos dados.
