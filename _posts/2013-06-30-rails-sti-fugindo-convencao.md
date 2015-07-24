---
title: "Rails STI: Fugindo da convenção"
author: Guilherme Ceolin
date: 2013-06-30
---


O suporte a STI ([Single Table Inheritance](http://http://en.wikipedia.org/wiki/Single_Table_Inheritance)), assim como tudo no rails, segue uma convenção. Mas e quando precisamos sair dela?

Antes de começar, fica um recado: Não seguir as convenções é a pior coisa a se fazer usando rails.

O framework foi construido em cima de convenções, fugir delas da trabalho, e requer bastante conhecimento do que se está fazendo, para nao atrapalhar o bom funcionamento do mesmo.

Dito isso, vamos lá.

Alguns dias atras me deparei com o seguinte problema: precisei extrair alguns models de uma aplicação para uma engine. Porem, como é um projeto em produção, com uma grande massa de dados, com o banco de dados compartilhado com outra aplicação, tive que tomar alguns cuidados.

Para quem não está familiarizado com Rails Engines, sugiro ler o [Rails Guides](http://edgeguides.rubyonrails.org/engines.html) sobre o assunto.

Nesta app, é utilizado STI para definir papeis na hora de autenticar usuarios. ex:

```ruby
class User < ActiveRecord::Base
...
end

class Professor < User
...
end

class Student < User
...
end
```

Ao extrair o modulo para uma engine chamada `Authentication`, estas mesmas classes ficaram assim:

```ruby
# app/models/authentication/user.rb
module Authentication
  class User < ActiveRecord::Base
  ...
  end
end

# app/models/authentication/professor.rb
module Authentication
  class Professor < ActiveRecord::Base
  ...
  end
end

# app/models/authentication/student.rb
module Authentication
  class Student < ActiveRecord::Base
  ...
  end
end
```

### Problema

Todos os registros da tabela `users` estão com o campo `type` preenchidos com o nome das classes sem o namespace.

Portanto, ao tentar recuperar um usuário, vamos ter problemas:

```bash
# Antes da alteração:
irb(main):001:0> Student.first
  Student Load (0.3ms)  SELECT "users".* FROM "users" WHERE "users"."type" IN ('Student') LIMIT 1
=> #<Student id: 2, name: "Anakin", email: "vader@empire.com", password: "[SECRET]", type: "Student">
irb(main):002:0> Professor.first
  Professor Load (0.3ms)  SELECT "users".* FROM "users" WHERE "users"."type" IN ('Professor') LIMIT 1
=> #<Professor id: 1, name: "Yoda", email: "yoda@jedi.com", password: "[SECRET]", type: "Professor">

# Apos a alteração:
irb(main):001:0> Authentication::User.all
  Authentication::User Load (0.2ms)  SELECT "users".* FROM "users"
ActiveRecord::SubclassNotFound: The single-table inheritance mechanism failed to locate the subclass: 'Professor'. This error is raised because the column 'type' is reserved for storing the class in case of inheritance. Please rename this column if you didn`t intend it to be used for storing the inheritance class or overwrite Authentication::User.inheritance_column to use another column for that information.
   (backtrace ommited)
irb(main):002:0> Authentication::Professor.first
  Authentication::Professor Load (0.3ms)  SELECT "users".* FROM "users" WHERE "users"."type" IN ('Authentication::Professor') LIMIT 1
=> nil
```

Vemos que o rails nao consegue encontrar a classe `Professor`, porque agora ela está dentro de um namespace: `Authentication`.

E mais: vemos que, mesmo chamando diretamente a classe `Authentication::Professor`, nenhum registro é encontrado no banco.

### Solução

Depois de muito pesquisar, acabei chegando a um metodo chamado `sti_name` definido [aqui](https://github.com/rails/rails/blob/master/activerecord/lib/active_record/inheritance.rb#L97-L99). Como vocês podem ver, este metodo verifica uma opção chamada `store_full_sti_class`, cujo padrão `true` é definido [aqui](https://github.com/rails/rails/blob/master/activerecord/lib/active_record/inheritance.rb#L8-L10).

Não encontrei nenhuma documentação sobre o essa opção, somente o comentario escrito no codigo:

```ruby
  # from https://github.com/rails/rails/blob/master/activerecord/lib/active_record/inheritance.rb
  # Determines whether to store the full constant name including namespace when using STI.
  class_attribute :store_full_sti_class, instance_writer: false
  self.store_full_sti_class = true
```

Em outras palavras, essa opção define se o Namespace será usado ou não, pra definir o campo `type` em uma STI.

Bastou setar essa opção para `true`, que tudo funcionou como esperado:

```ruby
module Authentication
  class User
    self.store_full_sti_class = false
  ...
  end
end
```
E com isso, conseguimos recuperar novamente os dados da tabela `users`:

```bash
irb(main):001:0> Authentication::User.all
  Authentication::User Load (0.1ms)  SELECT "users".* FROM "users"
=> [#<Authentication::Professor id: 1, name: "Yoda", email: "yoda@jedi.com", password: "[SECRET]", type: "Professor">, #<Authentication::Student id: 2, name: "Anakin", email: "vader@empire.com", password: "[SECRET]", type: "Student"]
irb(main):002:0> Authentication::Professor.first
  Authentication::Professor Load (0.3ms)  SELECT "users".* FROM "users" WHERE "users"."type" IN ('Professor') LIMIT 1
=> #<Authentication::Professor id: 1, name: "Yoda", email: "yoda@jedi.com", password: "[SECRET]", type: "Professor">
```

Uma dica final: Só encontrei essa opção após ler o codigo [fonte do rails](https://github.com/rails/rails). Aconselho que todos façam isso, é uma otima maneira de aprender sobre o framework, e também sobre padrões de codigo.

Obrigado, e até a próxima!
