---
title: Ações
type: guide
order: 3
---

## O que é uma ação?

As ações são os _building blocks_ do Stellar, está é a unidade básica da Framework. Sendo o Stellar uma Framework baseada em ações, isto significa que existe um repositório com todas as ações registadas no projeto. Uma ação representa uma pequena funcionalidade do projeto, elas podem ser chamadas diretamente pelo cliente ou então por outras ações. As ações podem receber um conjunto de _inputs_ que depois de processados devolvem um conjunto de _outputs_. Estas ações podem ser privadas, podendo apenas serem chamadas por outras ações e não pelo cliente e também podem ser sobrescritas por outros módulos, amenos que se encontrem protegidas contra isso.
```javascript
exports.randomNumber = {
    name: 'randomNumber',
    description: 'Generate a random number',
    outputExample: {
        number: 0.40420848364010453
    },

    run: function(api, data, next) {
        // generate a random number
        var number = Math.random()

        // save the generated number on the response property
        data.response.number = number

        // return a formated string
        data.response.formatedNumber = 'Your random number is ' + number

        // finish the action execution
        next()
    }
}
```

As ações são compostas por duas propriedades obrigatórias, uma é a identificação da ação (`name`) e a outra é a lógica (`run`) da ação, mas esta pode conter muitas mais informações adicionais tal como uma descrição, restrições aos valores de _input_, _middleware_ e um exemplo de _output_. Com esta meta informação o Stellar é capaz de gerar documentação de forma totalmente automática sem intervenção humana. Isto é excelente para grandes equipas de forma a que todos os elementos possam de forma fácil ficar a saber de todas as funcionalidades do projeto sem terem que perguntar a outros elementos da equipa. Na figura acima pode-se ver a estrutura de uma ação, esta ação é responsável por gerar uma numero aleatório.

Existe um conjunto de opções a que podem ser adicionadas as ações, abaixo encontram-se enumeradas e descritas todas as opções disponíveis.

* `name`: Identificador único da ação;
* `description`: Descreve de forma extensa a ação (o seu objetivo), esta informação é importante para obter a documentação automática;
* `inputs`: Enumera os parâmetros de entrada da ação. Também é possível aplicar restrições aos valores de _input_;
* `middleware`: Indica os _middlewares_ a serem aplicados antes e depois da execução da ação. Os _middlewares_ globais são automaticamente aplicados;
* `outputExample`: Contem um exemplo de uma resposta da ação. Este exemplo será anexado automaticamente à documentação gerada pelo Stellar;
* `blockedConnectionTypes`: Permite bloquear tipos de conexões na ação que está a ser definida;
* `logLevel`: Permite definir como a ação deve ser registada;
* `protected`: Quando `true` impede que a ação seja subscrita por um módulo de maior prioridade;
* `private`: A ação apenas pode ser chamada internamente;
* `toDocument`: Por defeito esta opção está definida para `true`, caso contrario não será gerada documentação para esta ação;
* `run`: Por fim, a lógica da ação, trata-se de uma função composta por três parâmetros de entrada (api, action, next).

O Stellar suporta múltiplas versões da mesma ação. Isto permite suportar ações com o mesmo nome, mas com funcionalidades melhoradas. Está funcionalidade é bastante útil quando existem muitas aplicações cliente a se alimentar da API e pode-se assim atualizar cada aplicação individualmente para a nova API sem interrupção do serviço nas demais.

As ações podem conter opcionalmente o parâmetro `version` para definir a versão da mesma. A quando o pedido do cliente pode-se usar o parâmetro `apiVersion` para pedir uma versão especifica da ação.

> Nota-se que quando não é especificado o parâmetro `apiVersion` o Stellar irá responder com a ultimas versão da ação.

## Declaração de Inputs

Na declaração das ações podem ser indicados os campos de _input_ recorrendo à propriedade `inputs`, isto fará com que sejam aplicadas restrições aos dados de entrada. Estas restrições podem ser validators já existentes no sistema, uma expressão regular ou uma função que retorna um _boolean_ (em que `true` indica que o valor de _input_ é valido).

A lista a baixo mostra as opções disponíveis para a declaração dos _inputs_:

* `required`: Este campo informa se o parâmetro é obrigatório;
* `convertTo`: Permite converter o parâmetro para um dado tipo de dados ou formato;
* `default`: Valor por defeito, caso o parâmetro não esteja presente no conjunto de _inputs_ na chamada co cliente;
* `validator`: Valida o parâmetro conta uma ou um conjunto de restrições.

## Converter Parametros

Para remover a necessidade dos desenvolvedores converterem manualmente os parâmetros o Stellar implementa uma forma de o fazer automaticamente antes da execução da ação. A propriedade `convertTo` pode ser uma _string_ com os valores (string, integer ou float) ou uma função.

### Exemplo

O exemplo abaixo mostra a convenção de um parâmetro para o tipo `Number`:

```javascript
exports.giveAge = {
  name: 'giveAge',
  description: 'Give the name and age based on the year of birth',

  inputs: {
    name: {
      required: true
    },
    birthYear: {
      required: true,
      convertTo: ‘integer’
    }
  },

  run: (api, action, next) => {
    // calculate the person age (birthYear is already a number)
    let age = new Date().getFullYear() - action.params.birthYear

    // return a phrase with the name and age
    action.response.result = `${action.params.name} has ${age}`

    // finish the action execution
    next()
  }
}
```

## Parâmetro action

O segundo parâmetro da função run, o objeto `data`, guarda o estado da conexão no momento em que a ação é chamada, neste momento os _midlewares_ de pré processamento já foram executados e os valores de _input_ validados.

## Chamadas Internas

Com vista a melhorar o reaproveitamento de código e fazer uma melhor separação das ações que partilham parte da mesma lógica, o Stellar implementa um mecanismo que permite fazer chamadas internas a ações. Isso quer dizer que se pode extrair parte da lógica de uma (ou mais) ações para ações mais simples, podendo essa mesma lógica ser usada por outras ações. Assim, a partir da composição de ações simples pode-se criar ações mais complexas sem tornar a leitura do código mais difícil ou dificultar a mantenabilidade das aplicações e módulos.


Para chamar uma ação internamente recorresse ao método `api.actions.call(actionName, params, callback)`:

* `actionName`: Nome da ação a ser chamada;
* `params`: Parâmetros a serem passados à ação que vai ser executada;
* `callback(error, response)`: função de __callback__:
  * `error`: erro devolvido pela ação chamada;
  * `response`: objeto com a resposta da ação chamada.

### Ações Privadas

Por vezes serão criadas ações que os desenvolvedores não querem que sejam chamadas pelos clientes, porque são apenas para uso interno, não realizam nenhuma operação relevante para o cliente ou são ações mais simples ou criticas que não devem ter exposição publica. Para isso o Stellar permite definir uma ação como privada, fazendo com que esta apenas possa ser chamada internamente. Para tornar uma ação privada basta adicionar a propriedade `private: true` na ação em questão.

### Exemplo

O exemplo abaixo mostra a chamada interna da ação ’sumANumber’, após a execução da ação é apresentado o resultado na consola. O exemplo completo pode ser encontrado [aqui](https://github.com/gil0mendes/stellar/blob/dev/example/modules/test/actions/internalCalls.js).

```javascript
api.actions.call('sumANumber', {a: 3, b: 3}, (error, response) => {
  console.log(`Result => ${response.formatted}`)
})
```

> NOTA: Também é possível chamar ações nas tarefas.

## Documentação Automatica

O Stellar permite gerar documentação das ações de forma completamente automática. A informação necessária é extraída através da declaração das propriedades das ações. Para fazer com que a não seja gerada uma página de documentação para uma data ação adiciona-se a propriedade `toDocument: false` na ação em questão, caso queira desativar para todas as ações define-se a configuração `api.config.general.generateDocumentation` para `false`. Para aceder à documentação basta visitar o endereço `docs/index.html` no endereço do servidor HTTP.

## Middlewares

É possível aplicar middlewares nas ações (antes e depois da execução das mesmas). Os middlewares podem ser globais (aplicados a todas as ações) ou locais, especificamente para cada ação através da propriedade `middleware` , fornecendo o nome de cada middleware a serem usados.

Pode aprender mais sobre middlewares (aqui)[middleware.html].

### Exemplo

O exemplo abaixo mostra a declaração de uma ação que contem dois middlewares:

```javascript
exports.getAllAccounts = {
  name: 'getAllAccounts',
  description: 'Get all registered accounts',

  middleware: ['auth', 'superResponse'],

  run: (api, action, next) => {
    // perforce some operation...

    // finish action execution
    next()
  }
}
```