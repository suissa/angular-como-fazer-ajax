# Como criar uma requisição AJAX do jeito certo com AngularJS?

Estou escrevendo esse post pq em 1 dia 2 pessoas me pediram ajuda sobre esse tema, então bora lá escrever para todos de uma vez. :p

## Módulo

Primeiramente você deverá criar um `module` da nossa aplicação:

```js
angular.module('Webschool', [])
```

Para criarmos um módulo com o Angular basta passarmos 2 parâmetros para a função `angular.module`:

- nome do módulo: 'Webschool';
- array de depêndencias: []. //nessa caso vazio

Após criamos o módulo precisamos criar um `Controller` que fará nossa requisição, para isso podemos encadear a função `controller` tanto na criação do `module` como na sua seleção.

**HEIN!!?? Como assim seleção do `module`?**

É o seguinte, você pode criar módulos no Angular de qualquer lugar, porém para que você possa reusá-lo deverá apenas chamar ele pelo seu nome, desse jeito:

```js
var app = angular.module('Webschool');
// app.controller();
```

Percebeu que dessa vez eu não passo o segundo parâmetro?

Isso serve para eu selecionar o módulo `Webschool` e utilizá-lo na variável `app`, porém como eu sigo o [Style Guide do John Papa](https://github.com/johnpapa/angular-styleguide) eu não jogo o módulo nessa variável, podendo usá-lo apenas assim:

```js
angular.module('Webschool');
// angular.module('Webschool').controller()
;
```

Pois desse jeito você não perde qual referencia você tem em `app`, sabendo sempre qual módulo esta usando, pois sempre chamará-o pelo seu nome.

## Controller

Então para eu usar o módulo selecionado eu só preciso encadear as funções desejadas, por exemplo um `controller`:


```js
angular.module('Webschool', [])
.controller('GithubController', ['$scope', '$http',
  function($scope, $http){
  }
]);
```

Na função `controller` também passamos 2 parâmetros:

- nome do Controller: `GithubController`
- array de dependências e sua função: `['$scope', '$http', function($scope, $http){}]`

Porém antes de continuarmos vamos colocar no estilo do John Papa, retirando a função do **array** de dependências e injetando as dependências de forma separada, primeiramente vamos retirar a função daqui:

```js
angular.module('Webschool', [])
.controller('GithubController', ['$scope', '$http', GithubController]);

function GithubController($scope, $http) {};
```

Agora retiramos as dependências, colocando-as como um **array** de *Strings* no atributo `$inject` do nosso `Controller`:

```js
angular.module('Webschool', [])
.controller('GithubController', GithubController);

function GithubController($scope, $http) {};
GithubController.$inject = ['$scope', '$http'];
```

PRONTOOOOO!!

## Ajax

Agora podemos começar a programar como gente, vamos criar agora nossa requisição *Ajax* utilizando o módulo `$http`, inicialmente criamos um objeto da requisição que será injetado na função `$http`:

```js
angular.module('Webschool', [])
.controller('GithubController', GithubController);

function GithubController($scope, $http) {

  var REQ = {
    url: 'https://api.github.com/users/suissa',
    method: 'GET'
  };

  $http(REQ)
    .success(function(data){
      console.log('Data: ', data);
    })
    .error(function(err){
      console.log('Erro: ', err);
    });
}
GithubController.$inject = ['$scope', '$http']
```

Perceba então que após o `$http()` nós estamos encadeando as funções:

- success
- error

Que na verdade são atalhos para os *callbacks* da *[Promise](http://nomadev.com.br/angularjs-promises-promessas-o-guia-definitivo/)* do `$http`.

A função `$http` sempre retornará uma *Promise* com 2 funções, uma será executada apenas se a requisição for `OK`, senão ela executará a outra, que é função de erro.

```js
angular.module('Webschool', [])
.controller('GithubController', GithubController);

function GithubController($scope, $http) {
  var REQ = {
    url: 'https://api.github.com/users/suissa',
    method: 'GET'
  };
  $http(REQ)
    .success(function(data){
      console.log('Data: ', data);
      $scope.User = data;
    })
    .error(function(err){
      console.log('Erro: ', err);
    });
};
GithubController.$inject = ['$scope', '$http'];
```

Com isso já estamos recebendo os dados do meu usuário no [github.com](https://github.com/suissa/) e precisamos apenas mostrá-los usando esse código bem simples no nosso template:


```html
<p data-ng-controller='GithubController'>
  <img class='user-avatar' data-ng-src="{{user.avatar_url}}" alt="">
  <br />
  <span class='user-label'>Login:</span> {{User.login}} 
  <br />
  <span class='user-label'>Name:</span> {{User.name}}
  <br />
  <span class='user-label'>Company:</span> {{User.company}}
  <br />
  <span class='user-label'>Blog:</span> {{User.blog}}
  <br />
  <span class='user-label'>Email:</span> {{User.email}}
  <br />
  <span class='user-label'>Location:</span> {{User.location}}
</p>
```

Rodando nosso esse código no `index.html` teremos algo assim:

![Screenshot do resultado](https://cldup.com/j4VvOPOGlt-3000x3000.png)

### Porém, ESSE NÃO É O JEITO CERTO!!

Calma, estamos quase lá.

## Service

**Mas o porquê que não está certo?**

Basicamente porque é trabalho do `Service` encapsular esse tipo de acesso à dados, nesse caso usando o `$http`, então vamos refatorar nosso código:

```js
angular.module('Webschool', [])
.controller('GithubController', GithubController)
.service('GithubService', GithubService);

function GithubController($scope, GithubService) {
  GithubService.getUser("suissa")
    .success(function(data){
      console.log('Data: ', data);
      $scope.User = data;
    })
    .error(function(err){
      console.log('Erro: ', err);
    });
};

function GithubService($http) {
  var REQ = {
    url: 'https://api.github.com/users/',
    method: 'GET'
  };

  this.getUser = function(userLogin) {
    // recebo o login da busca e concateno na url da busca
    REQ.url += userLogin;
    return $http(REQ);
  };
}

// Dependencias
GithubController.$inject = ['$scope', 'GithubService'];
GithubService.$inject = ['$http'];

```


Agora sim está OK, separamos o `Controller` do `Service` onde cada um deve ter uma responsabilidade única, caso não conheça esse conceito deixo aqui para você um ótimo assunto para leitura: [SOLID](http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod).

**SOLID** é um acrônimo para:

- Single responsibility;
- Open-closed;
- Liskov substitution;
- Interface segregation;
- Dependency inversion.

Aconselho fortemente a leitura sobre esse tema.

Até a proxima!!! :*

