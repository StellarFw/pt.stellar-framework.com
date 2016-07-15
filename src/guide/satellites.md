---
title: Satélites
type: guide
order: 6
---

## O que é um Satellite?

O Engine por si só não faz absolutamente nada, a única lógica que este possui assim que a instancia do Stellar é iniciada é a de procurar pelos Satellites e carregar-los.

![Satellite Stages](/images/satellite_stages.png)

Como pode ser visto na figura acima é mostrada as três fazes de carregamento de um Satellite, são elas: _load_, _start_ e _stop_. A etapa de carregamento é obrigatória, enquanto a de inicialização e de paragem são opcionais. No caso de ser iniciada uma operação sem previsão de paragem, na etapa de inicialização, é recomendado efetuar a sua paragem na terceira etapa (_stop_), isto porque é possível reiniciar o servidor, sem que o processo de execução do Stellar tenha que ser terminado.

## Formato

Um Satellite deve ser uma classe escrita segundo o padrão [ES6](http://www.ecma-international.org/ecma-262/6.0/index.html). O único requisito para o Satellite ser carregado pelo Stellar é contem um método `load(api, next)`. Existem outras propriedades, que se encontram descritas abaixo:

* `loadPriority`: Permite alterar a ordem de carregamento do Satellite, o valor por defeito é 100;
* `startPriority`: Permite alterar a ordem de inicio do Satellite, o valor por defeito é 100;
* `stopPriority`: Permite alterar a ordem de paragem, o valor por defeito é 100;
* `load(api, next)`: Operação a ser executada a quando o carregamento do Satellite;
* `start(api, next)`: Operação a ser executada a quando o inicio do Satellite;
* `stop(api, next)`: Operação a ser executada quando o Satellite for terminado.

## Exemplo

```javascript
'use strict'

/**
 * Satellite class.
 *
 * It's recommended use this class only to specify the Satellite function, other
 * logic must be written in another class.
 */
exports.default = class {

    /**
     * Satellite constructor.
     *
     * The developer must define the priority order for the Satellite's stages
     * here.
     */
    constructor () {
      // define satellite load priority.
      this.loadPriority = 10
    }

    /**
     * Satellite loading function.
     *
     * @param  {{}}}      api  API object reference.
     * @param  {Function} next Callback function.
     */
    load (api, next) {
      // log an example message
      api.log('This is awesome!', 'info')

      // finish the satellite load
      next()
    }

}
```