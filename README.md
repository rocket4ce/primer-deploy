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


Y listo!!! tenemos nuestra mega aplicación, por ahora dejaremos nuestra super app para configurar nuestro servidor.

Asumo que tienen instalado VirtualBox y instalada nuestra image de ubuntu server.
Estoy usando ubuntu 16.04.1 server.

La configuración para conectarse via ssh a nuestro server corriendo en virtualbox, tienen que configurarlo como adaptador de puente.

 ![enter image description here](https://lh3.googleusercontent.com/-q9kcT0bmgSQ/WBwuQl6rfmI/AAAAAAAAA4s/DSC2Pquk4BU_3MxCclH4yVb825uzWKbQwCLcB/s0/adaptador-puente.png "adaptador-puente.png")

luego para saber cual es nuestra ip de nuestro servidor escribirmos en nuestra terminal de virtualbox  `ifconfig`

![enter image description here](https://lh3.googleusercontent.com/-lyT4taUpnEE/WBwvFr_s-XI/AAAAAAAAA44/NMpsgreMm58Q7cVW2dJSVzOg1KT_pU4pQCLcB/s0/ifconfig.png "ifconfig.png")

Luego entramos a nuestra terminal y nos conectamos vía shh.

ssh nombre-usuario@escribimos-nuestra-ip
el mío quedo algo así

    ssh rocket@192.168.1.43

   ![enter image description here](https://lh3.googleusercontent.com/-HSQ7lD5dGj0/WBwwFzRpXjI/AAAAAAAAA5I/7n2-eW1LN_cALcmn6yNHwb1NICC-XPA_QCLcB/s0/ssh.png "ssh.png")

Me pedira mi contraseña. y entrare.

![enter image description here](https://lh3.googleusercontent.com/-q7qErfrMy2U/WBwwcISqe8I/AAAAAAAAA5Q/uX1d0qNj1lMuo7OoyqTi4WBY17j5K0lJwCLcB/s0/conectados.png "conectados.png")

Y entramos!! wooo ahora se pone bueno!!... :D

*Para conectarse via ssh en windows usas **`PuTTY`**. Me acuerdo con poco éxito conectarme bien via windows, creo por falta de experiencia, pero recomiendo usar entorno linux. Es mucho mas fácil.*

Primero que nada creamos un usuario deploy, antes que la caguemos usando root. :P

    sudo adduser deploy
    sudo adduser deploy sudo
    su deploy
El primer comando crea un usuario llamado deploy.
El segundo de da poder sudo al usuario deploy.
Y por ultimo `su deploy` veremos que esta todo ok nos redirigira a nuestro usuario deploy y nos pedirá la contraseña que le asignamos.

Si todo sale bien estaremos con usuario deploy.  escribiremos `exit` para volver a root o el usuario que tienes asignado como root.

Este paso es opcional, pero a mi me gusta no tener que meter la contraseña cada vez que me conecto con ssh a mi server.
Para ellos me desconectare de mi servidor escribiendo de nuevo `exit` en la consola, y usaremos ssh-copy-id, esta herramienta nos permitirá conectarnos via keys a nuestro servidor sin pedirnos contraseña (esta opción es para mac, y linux, no tengo idea si existe para windows).

Para instalar en mac vía brew, escribiremos en nuestra terminal `brew install ssh-copy-id`. **Ojo esto lo tenemos que hacer en nuestra maquina local no en nuestro servidor.**

En linux creo que ya viene instalado.

Ya instalado escribiremos en nuestra terminal

    ssh-copy-id deploy@DIRECCIONIP
en mi caso es `ssh-copy-id deploy@192.168.1.43`, me pedirá ingresar la contraseña asignada el usuario deploy de nuestro servidor, la introducimos y listo ahora me podré conectar a mi servidor sin necesidad de meter contraseña.

ahora si entro a mi servidor como antes con el usuario deploy

    ssh deploy@192.168.1.43
no me pedirá contraseña!! :)
