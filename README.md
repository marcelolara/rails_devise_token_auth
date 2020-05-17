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
    
    
5. Inicializar el modelo del usuario para devise

    `rails generate devise_token_auth:install User auth`
    
    Una vez creada la migración, editar el archivo **db/migrate/[timestamp]_devise_token_auth_create_users.rb**
    modificando la plantilla inicial del usuario creada por devise para asignar UUID como generador del id en la
    cabecera de creación de la tabla.
    
    ```ruby
    create_table :users, id: :uuid do |t|
    ```
    
    Configurar el modelo del usuario para que sea extendido por devise, editando el archivo **app/models/user.rb**
    
    ```ruby
    class User < ActiveRecord::Base
     extend Devise::Models
    
     devise :database_authenticatable,
            :registerable,
            :recoverable,
            :rememberable,
            :trackable,
            :validatable
    
     include DeviseTokenAuth::Concerns::User
    end
    ```

    Debido a que queremos que el usuario que ingresa sea rastreable y además viene por defecto con el modelo de usuario
    de Devise es necesario generar la migración para extender el modelo y agregar las columnas faltantes, sino es muy
    probable que al momento de loguearse e rails explote.
    
    `rails g migration add_devise_trackable_columns_to_user`
    
    Editar el archivo de migración agregando las columnas para el rastreo.
    
    ```ruby
    class AddDeviseTrackableColumnsToUser < ActiveRecord::Migration[6.0]
      def change
        add_column :users, :sign_in_count, :integer, default: 0, null: false
        add_column :users, :current_sign_in_at, :datetime
        add_column :users, :last_sign_in_at, :datetime
        add_column :users, :current_sign_in_ip, :string
        add_column :users, :last_sign_in_ip, :string
      end
    end
    ```
   
    Ejecutar la migración
    
    `rails db:migrate`
   
   
    
    
    
    
    