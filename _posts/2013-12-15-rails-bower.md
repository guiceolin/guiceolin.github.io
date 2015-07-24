---
title: Rails + Bower = Awesome!
author: Guilherme Ceolin
date: 2013-12-15
--

Gerenciar dependencias de front-end com Rails não é facil.<br />
Se você quer colocar, por exemplo, o twitter bootstrap na sua app, normalmente há duas alternativas:

* Usar uma gem que o encapsule, e [existem](https://github.com/seyhunak/twitter-bootstrap-rails) [várias](https://github.com/anjlab/bootstrap-rails);
* Baixar o fonte da [pagina oficial](http://getbootstrap.com).

No primeiro caso, o maior problema é quando uma nova versão do bootstrap é lançada, que precisamos esperar o autor da gem atualiza-la para que suporte essa nova verão. Não chega a ser um problema se você não quizer usar sempre a versão mais nova, mas se você, como eu, vive no topo da onda, talvez seja um grande problema.

Então a solução é baixar do site oficial, e colocar na pasta `vendor/assets`, e corre pro abraço, certo? Não.<br />
Você precisa colocar os javascripts e css nas pastas corretas, `vendor/assets/javascripts` e `vendor/assets/stylesheets` respectivamente, e corrigir as chamadas `url()` do css.<br />
E isso a cada nova versão lançada.

A tristeza reina, até que você conhece o Bower.

###Bower

O [Bower](http://bower.io) é um gerenciador de dependencias para assets, algo preciso com o Bundler.<br />
Se você não sabe do que se trata, de uma olhada no site oficial.

Para integrar ele com o Rails, e principalmente com o Sprockets, é simples, mas precisa de alguns cuidados.<br/>
Sabemos que o sprockets carrega tudo que está no `vendor/assets/`, nas pastas de javascrips e stylesheets. Mas há uma pasta, `vendor/assets/components`, que serve justamente para guardar componentes (Duh!) com javascript e css juntos. Este é o caso do bootstrap, por exemplo.

Portanto, é preciso dizer ao bower para instalar os componentes nesta pasta expecifica. Para isso, crie um arquivo chamado `.bowerrc` na raiz do seu projeto, e adicione o seguinte a ele:

```json
{
  "directory": "vendor/assets/components"
}
```

E pronto. Agora pode requerer as libs normalmente no seu `application.js`

```javascript
//= require jquery
//= require jquery_ujs

//= require bootstrap
```

E no seu `application.css`:

```css
*= require bootstrap
```

Em alguns casos, como o [font-awesome](http://fontawesome.io) por exemplo, o arquivo .css está dentro outra pasta, portanto ficaria:

```css
*= require font-awesome/css/font-awesome.min
```

###jquery e jquery_ujs

Na instalação padrão do rails, o jquery e o jquery_ujs são carregados pela gem `jquery-rails`.<br />
O jquery costuma ser uma dependencia comum de biblitecas de assets, e provavelmente o bower resolverá e baixará uma versão que satisfaça todas as dependencias.<br />
Como o jquery_ujs ainda não foi atualizado para a versão 2.0.0 do jquery, pode ser que hajam problemas de incompatibilidade de versões, causando grande dor de cabeça.

Para resolver isso, podemos mover essa dependencia do jquery_ujs para o bower, declarando essa dependencia no nosso `bower.json`:

```javascript
{
  "name": "Awesome App",
  ...
  "dependencies": {
    "jquery": "*",
    "jquery-ujs": "*",
    ...
  },
  ...
}
```
Note que a lib chama-se `jquery-ujs` e não `jquery_ujs`. Portante lembre-se de corrigir o nome do import lá no `application.js`:

```javascript
//=require jquery
//=require jquery-ujs
...
```

E podemos remover o a dependencia da `jquery-rails` do `Gemfile`.

Pronto, agora nao dependemos mais de gens de terceiros ou de manter atualmente as dependencias de nossos assets!<br />
Viva o bower!


