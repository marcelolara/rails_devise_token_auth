# rails_devise_token_auth

Tutorial recordatorio para iniciar una API con Rails, Devise y Token_auth, UUID y PostgreSQL

## Pasos

1. Crear proyecto

    Inicializado como API, sin tests y postgreSQL como base de datos.

    `$ rails new rails_devise_token_auth --api -T -d postgresql`


2. Crear migración para habilitar la extensión de UUID en PostgreSQL

   `$ rails g migration EnableUuidExtension`

   En el archivo de migración agregar el código para habilitar pgcrypto.
    
   ```ruby
   class EnableUuidExtension < ActiveRecord::Migration[6.0]
     def change
       enable_extension 'pgcrypto'
     end
   end 
   ```
   Una vez modificado el archivo, generar la migración para que la extensión quede activa.
   
   `$ rails db:migrate`


3. Agregar al Gemfile las gemas necesarias

    Servirán para autentificar con devise y token_auth.

    ```ruby
    gem 'devise_token_auth'
    gem 'omniauth'
    ```
    Descomentar rack-cors para no tener dificultad de acceder a la API a través del navegador.

    ```ruby
    gem 'rack-cors', :require => 'rack/cors'
    ```

    Instalar las gemas.
    
    `$ bundle install`
    
    Configurar cors editando el archivo **config/initializers/cors.rb**
    reemplazando el contenido por este.
    
    ```ruby
    Rails.application.config.middleware.insert_before 0, Rack::Cors do
      allow do
        origins '*'
    
        resource '*',
                 headers: :any,
                 expose: ['access-token', 'expiry', 'token-type', 'uid', 'client'],
                 methods: [:get, :post, :put, :delete, :options,]
      end
    end
    ```


4. Configurar la base de datos

    Considerando que ya se encuentra configurado el servidor postgreSQL, es necesario agregar las bases de datos con
    las que trabajaremos editando el archivo **config/database.yml**.
    
   ```yaml
   default: &default
     adapter: postgresql
     encoding: unicode
     pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
   development:
     <<: *default
     database: rails_devise_token_auth_development
     username: YOUR_CUSTOM_USERNAME
     password: YOUR_PASSWORD
   test:
     <<: *default
     database: rails_devise_token_auth_test
     username: YOUR_CUSTOM_USERNAME
     password: YOUR_PASSWORD
   ```
   
   Crear la base de datos.
   
   `rails db:create`
    
    