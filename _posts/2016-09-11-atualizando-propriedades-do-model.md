---
layout: post
title: "[Convention over configuration] Atualizando propriedades do model no Ember após requisição para a API"
description: "Como atualizar atributos de um model Ember após requisição para API?"
category: convention-over-configuration
modified: 2016-09-12T14:40:43-03:00
tags: [ember slug update properties backend api convention-over-configuration]
image:
  feature: rails.jpg
---

Algumas propriedades são geradas/modificadas pela aplicação backend após o processamento da requisição. Isso acontece quando nossos objetos tem atributos que dependem de outros. Exemplo disso é o **slug**, ou id amigável, que usa como base campos como nome ou apelido. Quando esses campos são alterados, o slug é atualizado.

Outro exemplo disso é quando temos **upload de arquivos**. Quando fazemos o upload, é a API que irá salvar e nos devolver uma URL para download.

Mas como podemos fazer com que o **Ember** entenda que essas propriedades foram modificadas e reflita em nosso objeto e em nossa tela?

## O que poderia ser feito usando Jquery?

```javascript
$.ajax({
   type: "POST",
   url: "/users/" + user.slug,
   dataType: "json",
   data: { user: user },
   success: function (data) {
     user.slug = data.slug;
   },
});
```

## Porque não podemos usar o mesmo aproach com Ember?

Um dos maiores benefícios do Ember, assim com o Ruby on Rails, é a convenção sobre configuração. E como para quase tudo, o Ember possui uma convenção para resolver isso.

Poderíamos usar a mesma lógica do retorno de uma requisição ajax com Jquery, mas isso faria com que o objeto não tivesse os valores novos refletidos na store. Seria como se estivessemos tentando modificar o objeto novamente quando na verdade o servidor que modificou ele e isso já poderia ser automaticamente refletido.

Fazendo isso também funções como `changedAttributes()` retornariam também os valores alterados pós request.

## O jeito Ember de resolver o problema

Por convenção, o Ember lê o payload da request e caso estiver no formato esperado ele altera automaticamente o(os) objetos em questão. Ou seja, para o payload abaixo em uma requisição POST de um usuário, o Ember irá colocar o id no usuário que salvamos:

##### Padrão json:api

```json
{"data": [
    {
      "type": "users",
      "id": "1"
    }
  ]
}
```

##### Padrão json

```json
{ "user": { "id": 93243 } }
```

Para o payload abaixo em uma requisição PUT/PATCH de um usuário, o Ember irá atualizar o slug do usuário que salvamos:

##### Padrão json:api

```json
{"data": [
    {
      "type": "users",
      "id": "1",
      "attributes": {
        "slug": "william"
      }
    }
  ]
}
```

##### Padrão json

```json
{ "user": { "id": 93243, "slug": "william" } }
```

Isso acontece de forma automática, quando chamamos `user.save()` os valores novos são refletidos no objeto `user` logo após a requisição.

## E se não pudermos modificar o payload da API?

Nesse caso teríamos que lutar um pouco contra a convenção do ember. Se funções como `changedAttributes()` não forem importantes podemos fazer a requisição e atualizar o attributo após:

```javascript
user.save().then(function(response) {
  user.slug = response.data.slug;
});
```

## Conclusão

Apesar da curva de aprendizado ser alta com tecnologias convention-over-configuration como Rails e Ember, entendendo os padrões faz com que coisas complexas e trabalhosas se tornem fáceis e rápidas de serem desenvolvidas. Mas lutar contra as convenções faz com que as coisas sejam mais difíceis que o normal e se pudermos devemos evitar.
