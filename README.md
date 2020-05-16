# rails_devise_token_auth

Tutorial recordatorio para iniciar una API con Rails, Devise y Token_auth, UUID y PostgreSQL

## Pasos

1. Crear proyecto

    Inicializado como API, sin tests y postgreSQL como base de datos.

    `rails new rails_devise_token_auth --api -T -d postgresql`

2. Crear migración para habilitar la extensión de UUID en PostgreSQL

   `rails g migration EnableUuidExtension`

   En el archivo de migración agregar el código para habilitar pgcrypto.
    
   ```ruby
   class EnableUuidExtension < ActiveRecord::Migration[6.0]
     def change
       enable_extension 'pgcrypto'
     end
   end 
   ```
   Una vez modificado el archivo, generar la migración para que la extensión quede activa.
   
   `rails db:migrate`