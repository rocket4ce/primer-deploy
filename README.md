Tutorial - Deploy rails app en Ubuntu Server con CAPISTRANO - STACK: RVM, Ruby 2.3.0, RAILS 5.0.0.1, CAPISTRANO 3 PUMA NGINX POSTGRESQL y GIT
=======
***Introducción:***
Antes que todo este tutorial va dirigido a todas las comunidades de programadores, profesionales, entusiastas y autodidactas. En especial a la comunidad de programadores de la IV REGIÓN DE CHILE llamada [IVDevs](http://ivdevs.com).

Para seguir este tutorial pueden ver el [video](http://youtube.com/asdasdasd)

***Temas a tocar:***

 1. Crear una aplicación rails con postgres como motor de base de datos.
 2. Hacer un CRUD básico,  instalar Devise.
 3. Instalar Gemas para el deploy y configurarlas.
 4. Correr nuestro servidor local con ubuntu server y virtual box (configurar nuestro servidor local para poder conectarnos por ssh).
 5. Configuración de roles de usuario para el deploy.
 4. Instalar y configurar dependencias:
	 5. rvm
	 6. ruby -v 2.3.0
	 7. rails -v 5.0.0.1
	 8. postgrest
	 9. git
	 9. nginx

Creo que es todo. Así que manos a la obra!!

## 1.- Creación de nuestra mega aplicación llamada: primer-deploy ##

Mis versiones para este tutorial son los siguientes.

Ruby -v 2.3.0
Rails -v 5.0.0.1
Git

Abriremos nuestra terminal y empezaremos con la creación de nuestro proyecto. Ojo tienes que tener instalada postgrest en tu maquina local para que funcione el comando. (el flag --database o -d postgresql)

    rails new primer-deploy --database=postgresql
o más corta.

     rails new primer-deploy -d postgresql
ingresamos a nuestro proyecto

    cd primer-deploy

Como nosotros somos profesionales, o queremos serlo, usaremos git para nuestro deploy, por lo tanto iniciamos git en nuestro proyecto. (Tienes que tener instalado Git).

    git init
Nos vamos a nuestro gestor de repositorios favorito, ya sea github, bitbucket, gitlab etc etc.. Iniciamos nuestra cuenta y creamos un repositorio.
Luego lo agregamos a nuestro proyecto `git remote add origin` .

    git add . && git commit -m "primer commit, nuestro primer deploy" && git push origin -u master

Agregamos nuestra gema devise en nuestro gemfile

    #otras gemas
    gem "devise"
    #otras gemas

nos vamos a nuestra terminar corremos, `bundle install`, luego corremos los siguientes y monotonos comandos para generar nuestro login, registro etc etc etc.

    rails generate devise:install
    rails generate devise User
    rails db:create
    rails db:migrate

y seguimos configurando devise.

1. Ensure you have defined default url options in your environments files. Here
     is an example of default_url_options appropriate for a development environment
     in **config/environments/development.rb:**  

    config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
In production, :host should be set to the actual host of your application.

2. Ensure you have defined root_url to *something* in your config/routes.rb.
     For example:

       root to: "home#index"
3. Ensure you have flash messages in app/views/layouts/application.html.erb.
     For example:

       <p class="notice"><%= notice %></p>
       <p class="alert"><%= alert %></p>
4. You can copy Devise views (for customization) to your app by running:

    rails g devise:views

Luego de hacer todo lo que nos dijo devise, creamos un scaffold.

    rails g scaffold Post title:string body:text user:references
Luego corremos nuestra migración `rails db:migrate`

Nos vamos a configurar nuestro controlador de Post, app/controllers/posts_controller.rb

Antes de todo ponemos que un usuario tiene que estar registrado para crear un post, agregamos un callback que nos da devise, `before_action :authenticate_user!`.

    class PostsController < ApplicationController
      before_action :authenticate_user!
      before_action :set_post, only: [:show, :edit, :update, :destroy]
      # ...

Luego configuramos los metodos new, create y nuestro metodo set_post. quedaría algo así.

      # GET /posts/new
      def new
        @post = current_user.posts.new
      end
      #......
      # POST /posts
      # POST /posts.json
      def create
        @post = current_user.posts.new(post_params)

        respond_to do |format|
          if @post.save
            format.html { redirect_to @post, notice: 'Post was      successfully created.' }
            format.json { render :show, status: :created, location: @post }
          else
            format.html { render :new }
            format.json { render json: @post.errors, status: :unprocessable_entity }
          end
        end
      end
      #......

Nuestro metodo privado

      private
        # Use callbacks to share common setup or constraints between actions.
        def set_post
          @post = current_user.posts.find(params[:id])
        end

Ahora nos vamos a nuestro modelo user.rb agregmos `has_many :posts`  quedaría así:

    class User < ApplicationRecord
      # Include default devise modules. Others available are:
      # :confirmable, :lockable, :timeoutable and :omniauthable
      devise :database_authenticatable, :registerable,
             :recoverable, :rememberable, :trackable, :validatable

             has_many :posts
    end


Por ultimo nos vamos a nuestras rutas, en la carpeta config/routes.rb y agregamos

    root to: "posts#index"


Y listo!!! tenemos nuestra mega aplicación.
