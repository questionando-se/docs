---
layout: default
title: Formato de arquivo JSON do Banco de Dados
---

## Breve introdução

Antes de dar início a uma breve explicação da estrutura dos arquivos **.json** utilizados para armazenar as questões no banco de questões, é importante dizer que tanto o **CLI** (veja abaixo o que é), quanto essa documentação, quanto o próprio banco de questões ainda está em desenvolvimento e, portanto, podem ter coisas que não são as melhores possíveis ou incompletas e o plano desse projeto é a longo prazo.

## Alguns termos

### CLI

**CLI** significa *command line interface*, ou seja, interface de linha de comando, e é, o **CLI** do questionando nada mais é que um programa, escrito em **Node** com o **TypeScript** que permite interpretar os dados dos arquivos de questões e gerar as páginas da respectiva disciplina.

## Nomenclatura

TODO: falar sobre a nomenclatura dos arquivos.

## Estrutura do arquivo

Com o atual *schema* de questões para o banco de dados, cada arquivo **.json** deve conter apenas um objeto, contendo uma única questão, de acordo com o *schema* abaixo.

```javascript
{
    "exam": string,
    "year": number,
    "tags": string[],
    "content": QuestionFileContent[],
    "alternatives": [
        QuestionFileContent[],
        ...
    ],
    "notebook": string,
    "application": string?,
    "number": number,
    "correct": number
}
```

### [obrigatório] exam

Identificador do vestibular (o termo *exam*, referindo-se a exame/prova foi escolhido como nome do campo), deve ser um texto (ou uma *string*, melhor dizendo) que remete ao conjunto de dados no **CLI**, no repositório **db/cli/data/exams.ts**. Os identificadores de vestibular que estão disponíveis para o uso no **CLI**, atualmente, são:

- "**ENEM**": Exame Nacional do Ensino Médio.
- "**ENEM_PPL**: Exame Nacional do Ensino Médio para Pessoas Privadas de Liberdade e Jovens sob Medida Socioeducativa.
- "**FUVEST**": Fundação Universitária para o Vestibular.

### [opcional|obrigatório] year

O campo *year* se refere ao ano de aplicação do vestibular e deve ser um número (mais corretamente, um número inteiro). O campo é opcional e caso não seja adicionado, um novo campo nas seções **Por ano de realização** deverá aparecer e listar as questões que não possuem ano cadastrado.

**Observação:** o campo é opcional, porém é extremamente recomendado que seja preenchido e, em alguns casos, mesmo não sendo requerido, pelo **CLI**, o campo pode ser exigido, como é o caso de provas como o **ENEM**.

### tags

TODO: escrever sobre o campo **tags**.

### [obrigatório] content

O campo *content* remete ao conteúdo da questão e tem como formato, um array de objetos que devem se caracterizar como a interface do *TypeScript* **QuestionFileContent**:

```typescript
interface QuestionFileContent {
    type: string;
    data: any;
    pathType?: 'relative' | 'absolute';
}
```

O campo **type** irá determinar que tipo de conteúdo cada item no *array* terá e o campo **data** terá o formato descrito por cada tipo de item. O campo **pathType** não é obrigatório e só será utilizado para o caso das imagens. Os tipos suportados, atualmente, são:

#### text

O item será renderizado como um parágrafo. Deste modo, o campo **data** deverá ser um texto (ou melhor dizendo, uma *string*). Veja o exemplo:

```json
"content": [
    { "type": "text", "data": "Para produzir a Hydrangea cor-de-rosa de maior valor comercial, deve-se preparar o solo de modo que \\(x\\) assuma" }
]
```

#### image

O item será renderizado como uma imagem. Dessa forma, o campo **data** deverá ser um texto (*string*) com o link da imagem. Esse link pode ser absoluto (uma *url*, de fato) ou pode ser relativo ao repositório **db**. O valor padrão é *absolute*, porém, não é recomendado o uso de imagens externas, devido ao fato de que as mesmas podem ser excluídas prejudicando a manutenção do banco de questões. 

No caso de imagens e tabelas, pode-se adicionar, também, o campo **credits** que será um texto (*string*) e que remete a ideia dos creditos da imagem. Veja o exemplo:

```json
"content": [
    { "type": "image", "pathType": "relative",  "data": "assets/enem2018/matematica/remo.png", "credits": "Disponível em: www.remobrasil.com. Acesso em: 6 dez. 2017 (adaptado)." }
],
```

**Observação:** o campo **credits** ainda não está sendo renderizado nas páginas do banco de questões, porém é importante adicioná-los para evitar conflitos com as próximas atualizações do **CLI**.

#### ul

O item será renderizado como uma *unurdered list*, ou lista não ordenada, que é o elemento **UL** do **HTML**. O parâmetro **data** deve ser um novo array de **QuestionFileContent** onde cada item será um dos marcadores. Veja o exemplo:

```json
"content": [
    { "type": "ul", "data": [
        { "type": "text", "data": "Urna A - Possui três bolas brancas, duas bolas pretas e uma bola verde;" },
        { "type": "text", "data": "Urna B - Possui seis bolas brancas, três bolas pretas e uma bola verde;" },
        { "type": "text", "data": "Urna C - Possui duas bolas pretas e duas bolas verdes;" },
        { "type": "text", "data": "Urna D - Possui três bolas brancas e três bolas pretas." }
    ] }
]
```

#### table

O item será renderizado como uma tabela. O campo **data** deve ser um *array* onde cada item será um *array* do objeto com a *interface* **TableData** do **CLI**, descrito a seguir. Cada **TableData** é uma célula da tabela.

```typescript
interface TableData {
    header?: boolean;
    span?: number;
    rowspan?: number;
    text: string;
}
```

O campo **header** indica se a célula é cabeçalho ou não, o campo **span** é o equivalente ao *colspan* do **HTML**, o **rowspan** é equivalente ao *rowspan* do **HTML**. O campo **text** é o texto da célula (atualmente, somente textos são suportados como conteúdo das tabelas).

No caso de imagens e tabelas, pode-se adicionar, também, o campo **credits** que será um texto (*string*) e que remete a ideia dos creditos da imagem. Veja o exemplo:

```json
"content": [
    { "type": "table", "data": [
        [
            { "header": true, "text": "Nome" },
            { "header": true, "text": "Altura (m)" }
        ],
        [{ "text": "Bruninho" }, { "text": "1,90" }],
        [{ "text": "Dante" }, { "text": "2,01" }],
        [{ "text": "Giba" }, { "text": "1,92" }],
        [{ "text": "Leandro Vissoto" }, { "text": "2,11" }],
        [{ "text": "Lucas" }, { "text": "2,09" }],
        [{ "text": "Murilo" }, { "text": "1,90" }],
        [{ "text": "Ricardinho" }, { "text": "1,91" }],
        [{ "text": "Rodrigão" }, { "text": "2,05" }],
        [{ "text": "Serginho" }, { "text": "1,84" }],
        [{ "text": "Sidão" }, { "text": "2,03" }],
        [{ "text": "Thiago Alves" }, { "text": "1,94" }],
        [{ "text": "Wallace" }, { "text": "1,98" }]
    ], "credits": "Disponível em: www.cbv.com.br. Acesso em: 31 jul. 2012 (adaptado)" }
]
```

**Observação:** o campo **credits** ainda não está sendo renderizado nas páginas do banco de questões, porém é importante adicioná-los para evitar conflitos com as próximas atualizações do **CLI**.



### [obrigatório] alternatives

O campo *alternatives* contém as alternativas da questão. Cada item é um *array* de **QuestionFileContent**. Assim, sua descrição se assemelha com o campo [content](#content). Veja um exemplo:

```json
"alternatives": [
    [
        { "type": "text", "data": "1,90." }
    ],
    [
        { "type": "text", "data": "1,91." }
    ],
    [
        { "type": "text", "data": "1,96." }
    ],
    [
        { "type": "text", "data": "1,97." }
    ],
    [
        { "type": "text", "data": "1,98." }
    ]
]
```

Perceba, no exemplo, que são 5 alternativas, sendo que cada uma, possui um único **QuestionFileContent** no *array*.

### [opcional] notebook

Usado para provas do gênero do **ENEM**, onde existem diversos modelos de provas, ou no caso do **ENEM** cores dos cadernos.

### [opcional] number

Número da questão na respectiva prova. Caso o campo *notebook* tenha sido preenchido com o modelo da prova, a questão deve corresponder ao modelo da prova específicado no campo. Por exemplo, os campos exemplo abaixo, indicam que é a questão 171 do caderno Amarelo.

```json
"notebook": "Amarelo",
"number": 171,
```

### [obrigatório?] correct

Indica o índice (começa em 0) da questão correta. Se o exemplo abaixo indica as alternativas A, B, C, D e E, então o índice 0 indicará que a alternativa A é correta, o índice 1 indicará que a alternativa B é correta, ... e o índice 4 indicará que a alternativa E é a correta.

```json
"alternatives": [
    [
        { "type": "text", "data": "1,90." }
    ],
    [
        { "type": "text", "data": "1,91." }
    ],
    [
        { "type": "text", "data": "1,96." }
    ],
    [
        { "type": "text", "data": "1,97." }
    ],
    [
        { "type": "text", "data": "1,98." }
    ]
]
```

### [opcional] application

O campo deverá ser utilizado para indicar casos de reaplicação, porém, ainda não foi completamente desenvolvido e não tem uso no **CLI**. A ideia é que o mesmo seja mudado para um número que indica a aplicação, por exemplo, se atribuir o valor 2 para a mesma, indicará que a prova foi a segunda aplicação daquele ano. Um exemplo disso pode ser visto em reaplicações do **ENEM**.

## Equações matemáticas

TODO: falar sobre equações matemáticas aqui.

## Exemplos

Exemplos de arquivos de questões completas podem ser encontrados no repositório [db/db](https://github.com/questionando-se/db/tree/95db3c6d32ab77207f2b01b60702609547c58f5c/db).
