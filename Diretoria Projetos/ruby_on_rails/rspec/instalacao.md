---
order: 2
icon: diamond
label: "Como instalar RSpec ?"
---

<!-- Araújo -->

1. Adicionar a gem no Gemfile:

```bash
# ./Gemfile
# acrescentar o seguinte trecho:

group :development, :test do
  gem 'rspec-rails'
end
```

2. Instalar as dependências:

```bash
bundle install
```

3. Configurar o RSpec:

```bash
rails generate rspec:install
```


