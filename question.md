---
layout: default
title: Formato de arquivo JSON do Banco de Dados
highlight: true
---

## Breve introdução

Antes de dar início a uma breve explicação da estrutura dos arquivos **.json** utilizados para armazenar as questões no banco de questões, é importante dizer que tanto o **CLI** (veja abaixo o que é), quanto essa documentação, quanto o próprio banco de questões ainda está em desenvolvimento e, portanto, podem ter coisas que não são as melhores possíveis ou incompletas e o plano desse projeto é a longo prazo.

## Alguns termos

### CLI

**CLI** significa *command line interface*, ou seja, interface de linha de comando, e é, o **CLI** do questionando nada mais é que um programa, escrito em **Node** com o **TypeScript** que permite interpretar os dados dos arquivos de questões e gerar as páginas da respectiva disciplina.

## Nomenclatura

Para cada banco de dados (nesse caso, melhor dizendo, para cada disciplina), cada arquivo deve ter um nome único que deve ser interpretado como uma espécie de identificador. Aqui está sendo utilizado o padrão **qXXXXX.json** para cada arquivo, onde **XXXXX** representa um número com zeros a esquerda. Esse número deve ser único para cada arquivo em cada banco (ou melhor dizendo, em cada disciplina).

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

### exam

Identificador do vestibular (o termo *exam*, referindo-se a exame/prova foi escolhido como nome do campo), deve ser um texto (ou uma *string*, melhor dizendo) que remete ao conjunto de dados no **CLI**, no repositório [**db/cli/data/exams.ts**](https://github.com/questionando-se/db/blob/master/cli/data/exams.ts). Os identificadores de vestibular que estão disponíveis para o uso no **CLI**, atualmente, são:

- "**ENEM**": Exame Nacional do Ensino Médio.
- "**ENEM_PPL**: Exame Nacional do Ensino Médio para Pessoas Privadas de Liberdade e Jovens sob Medida Socioeducativa.
- "**FUVEST**": Fundação Universitária para o Vestibular.

### year

O campo *year* se refere ao ano de aplicação do vestibular e deve ser um número (mais corretamente, um número inteiro). O campo é opcional e caso não seja adicionado, um novo campo nas seções **Por ano de realização** deverá aparecer e listar as questões que não possuem ano cadastrado.

**Observação:** o campo é opcional, porém é extremamente recomendado que seja preenchido e, em alguns casos, mesmo não sendo requerido, pelo **CLI**, o campo pode ser exigido, como é o caso de provas como o **ENEM**.

### tags

O campo *tags* é um *array* de strings com identificador de tags de conteúdos. Caso não queira adicionar tags de conteúdos na questão, deixe como um array vazio:

```json
"tags": []
```

Para encontrar as tags suportadas, atualmente, veja o arquivo [**db/cli/data/tags.ts**](https://github.com/questionando-se/db/blob/master/cli/data/tags.ts). Veja, também, um exemplo de utilização:

```json
"exam": "ENEM",
"year": 2018,
"tags": ["geometria-plana", "triangulos"]
```

### content

O campo *content* é obrigatório e remete ao conteúdo da questão e tem como formato, um array de objetos que devem se caracterizar como a interface do *TypeScript* **QuestionFileContent** (Veja abaixo). Deste modo, cada item deverá ter os campos *type* e *data*, podendo conter alguns campos não obrigatórios, como o *pathType* para imagens.

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
]
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

### alternatives

O campo *alternatives* é obrigatório e contém as alternativas da questão. Cada item é um *array* de **QuestionFileContent**. Assim, sua descrição se assemelha com o campo [content](#content). Veja um exemplo:

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

### notebook

O campo é opcional e deve ser usado para provas do gênero do **ENEM**, onde existem diversos modelos de provas, ou no caso do **ENEM** cores dos cadernos.

### number

O campo não é obrigatório e indica o número da questão na respectiva prova. Caso o campo *notebook* tenha sido preenchido com o modelo da prova, a questão deve corresponder ao modelo da prova específicado no campo. Por exemplo, os campos exemplo abaixo, indicam que é a questão 171 do caderno Amarelo.

```json
{
    "notebook": "Amarelo",
    "number": 171
}
```

### correct

O campo é obrigatório e indica o índice (começa em 0) da questão correta. Se o exemplo abaixo indica as alternativas A, B, C, D e E, então o índice 0 indicará que a alternativa A é correta, o índice 1 indicará que a alternativa B é correta, ... e o índice 4 indicará que a alternativa E é a correta.

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

### application

O campo não é obrigatório e deverá ser utilizado para indicar casos de reaplicação, porém, ainda não foi completamente desenvolvido e não tem uso no **CLI**. A ideia é que o mesmo seja mudado para um número que indica a aplicação, por exemplo, se atribuir o valor 2 para a mesma, indicará que a prova foi a segunda aplicação daquele ano. Um exemplo disso pode ser visto em reaplicações do **ENEM**.

## Equações matemáticas

Equações matemáticas podem ser inseridas utilizando a sintaxe da linguagem **LaTeX**. Os códigos devem ser inseridos entre **\\(** e **\\)** para equações em linha e entre **\\[** e **\\]** para equações em bloco. O renderizador de equações **LaTeX** utilizado no **Questionando** é o **KaTeX** e, para seu melhor aproveitamento, é comendado a leitura da página [**Supported Functions**](https://katex.org/docs/supported.html) da sua documentação.

## Exemplos

Exemplos de arquivos de questões completas podem ser encontrados no repositório [db/db](https://github.com/questionando-se/db/tree/master/db).

### matematica/enem/2019/q00045.json

```json
{
    "exam": "ENEM",
    "year": 2019,
    "tags": [],
    "content": [
        { "type": "text", "data": "Uma empresa confecciona e comercializa um brinquedo formado por uma locomotiva, pintada na cor preta, mais 12 vagões de iguais formato e tamanho, numerados de 1 a 12. Dos 12 vagões, 4 são pintados na cor vermelha, 3 na cor azul, 3 na cor verde e 2 na cor amarela. O trem é montado utilizando-se uma locomotiva e 12 vagões, ordenados crescentemente segundo suas numerações, conforme ilustrado na figura." },
        { "type": "image", "pathType": "relative",  "data": "assets/enem2019/matematica/trem.png" },
        { "type": "text", "data": "De acordo com as possíveis variações nas colorações dos vagões, a quantidade de trens que podem ser montados, expressa por meio de combinações, é dada por" }
    ],
    "resolutions": [],
    "alternatives": [
        [
            { "type": "text", "data": "\\( C_{12}^{4} \\times C_{12}^{3} \\times C_{12}^{3} \\times C_{12}^{2} \\)." }
        ],
        [
            { "type": "text", "data": "\\( C_{12}^{4} + C_{8}^{3} + C_{5}^{3} + C_{2}^{2} \\)." }
        ],
        [
            { "type": "text", "data": "\\( C_{12}^{4} \\times 2 \\times C_{8}^{3} \\times C_{5}^{2} \\)." }
        ],
        [
            { "type": "text", "data": "\\( C_{12}^{4} + 2 \\times C_{12}^{3} + C_{12}^{2} \\)." }
        ],
        [
            { "type": "text", "data": "\\( C_{12}^{4} \\times C_{8}^{3} \\times C_{5}^{3} \\times C_{2}^{2} \\)." }
        ]
    ],
    "notebook": "Amarelo",
    "number": 137,
    "correct": 4
}
```

### matematica/enem/2019/q00062.json

```json
{
    "exam": "ENEM",
    "year": 2019,
    "tags": [],
    "content": [
        { "type": "text", "data": "O slogan “Se beber não dirija”, muito utilizado em campanhas publicitárias no Brasil, chama a atenção para o grave problema da ingestão de bebida alcoólica por motoristas e suas consequências para o trânsito. A gravidade desse problema pode ser percebida observando como o assunto é tratado pelo Código de Trânsito Brasileiro. Em 2013, a quantidade máxima de álcool permitida no sangue do condutor de um veículo, que já era pequena, foi reduzida, e o valor da multa para motoristas alcoolizados foi aumentado. Em consequência dessas mudanças, observou-se queda no número de acidentes registrados em uma suposta rodovia nos anos que se seguiram às mudanças implantadas em 2013, conforme dados no quadro." },
        { "type": "table", "data": [
            [
                { "header": true, "text": "Ano" },
                { "header": true, "text": "2013" },
                { "header": true, "text": "2014" },
                { "header": true, "text": "2015" }
            ],
            [{ "text": "Número total<br/>de acidentes" }, { "text": "1 050" }, { "text": "900" }, { "text": "850" }]
        ] },
        { "type": "text", "data": "Suponha que a tendência de redução no número de acidentes nessa rodovia para os anos subsequentes seja igual à redução absoluta observada de 2014 para 2015." },
        { "type": "text", "data": "Com base na situação apresentada, o número de acidentes esperados nessa rodovia em 2018 foi de" }
    ],
    "resolutions": [],
    "alternatives": [
        [
            { "type": "text", "data": "150." }
        ],
        [
            { "type": "text", "data": "450." }
        ],
        [
            { "type": "text", "data": "550." }
        ],
        [
            { "type": "text", "data": "700." }
        ],
        [
            { "type": "text", "data": "800." }
        ]
    ],
    "notebook": "Amarelo",
    "number": 153,
    "correct": 3
}
```

### Outras

Para mais exemplos, veja o repositório [db/db](https://github.com/questionando-se/db/tree/master/db).
