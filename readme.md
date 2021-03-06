## Sistema de delivery com Laravel 5.1 + Ionic

Roteiro de atividades realizadas no curso Laravel 5.1 + Ionic da CodeEducation. 
Criado para fixar meu aprendizado e servir como referências futuras.

###Capítulo 1: Criando a base do sistema

1. Gerar APP_KEY caso não tiver sido gerada na instalação do laravel
  ```
    php artisan key:generate
  ```
2. Definir variáveis no .env
3. Mudar nome da aplicação 
  ```
    php artisan app:name [namespace]
  ```
4. Criar database
5. Em relação aos Models:
  - Criada pasta app\Models 
  - User transferi para nova paste e namespace corrigido (no config\auth.php também)
  - Category, Product, Client, Order, OrderItem models criados
6. Em relação às Migrations criadas:
  - create_categories_table
  - create_products_table
  - create_clients_table
  - create_orders_table
  - create_order_items_table
7. Em relação às Factories criadas:
  - Category
  - Product
  - Client
8. Em relação aos Seeds:
  - UserTableSeeder (Client sendo criado junto)
  - CategoryTableSeeder
  - ProductTableSeeder (não utilizado, ver CategoryTableSeeder)
9. Relacionamentos:
  - Category hasMany Product
  - Product belongsTo Category
  - User hasOne Client
  - Client hasOne User
  - Order hasMany OrderItem
  - Order belongsTo User
  - Order hasMany Product
  - OrderItem belongsTo Product
  - OrderItem belongsTo Order
  
###Capítulo 2: Repositories (padrão de projeto)

1. https://github.com/andersao/l5-repository
  - php artisan vendor:publish
  - Arrumar repository.php rootNamespace e models
  - Recriar todos os models já criados no Cap1 utilizando o make:repository
2. Criar Provider
  - Criar RepositoryServiceProvider (php artisan make:provider)
  - Fazer bind de todos os repositorios criados
  ```php
  /* dentro de register() */
  $this->app->bind(
  	'CodeDelivery\Repositories\CategoryRepository',
  	'CodeDelivery\Repositories\CategoryRepositoryEloquent'
  );
  ```
3. Adicionar repository criado a lista de providers (app.php)

###Capítulo 3: Sistema Administrativo

1. Instalar packages
  - https://github.com/bestmomo/scafold
    1. Adicionar ao composer.json
    ```
      "minimum-stability": "dev",
    ```
	2. composer require bestmomo/scafold
	3. Adicionar o Bestmomo\Scafold\ScafoldServiceProvider::class a lista de service provider
	4. php artisan vendor:publish
  - https://github.com/illuminate/html
	1. composer require illuminate/html
	2. Adicionar o Illuminate\Html\HtmlServiceProvider::class a lista de service provider
	3. Adicionar os Facades 
	```php
	  /* dentro de 'providers' em app.php */
      'Html'      => Illuminate\Html\HtmlFacade::class,
      'Form'      => Illuminate\Html\FormFacade::class,
	```
2. Controllers
  - CategoryController
  - ProductsController
3. Views
  - admin.categories
  - admin.products
4. Paginação
5. Rotas nomeadas 
6. Custom Requests
  - AdminCategoryRequest
  - AdminProductRequest
7. Refatorando Forms (_form)
8. Agrupando rotas
9. Middleware CheckRole (admin criado nas seeds)

###Capítulo 4: Clientes

1. ClientsController
2. AdminClientRequest
3. Rotas de admin.clients
4. Views de clients
5. CRUD Clients
6. ClientService
  - Recebe os repositorios de Cliente e de Usuário como parâmetros em seu construtor
  - create e update, utilizado para criar um usuário automaticamente com uma senha padrão antes de executar o create e o update do repository
  
###Capítulo 5: Pedidos

1. OrdersTableSeeder
  - Colocar na ModelFactory Order e OrderItem
2. OrdersController
3. Criar views admin.orders.index e edit
4. Relação de Order com Client
  - OrderBelongsToClient
5. Seed para deliveryman
6. Função getDeliveryman no UserRepository para criar select de deliveryman

###Capítulo 6: Checkout

1. php artisan make:repository Cupom
2. php artisan make:migration create_cupoms_table --create=cupoms
  - atenção: adicionar foreign key cupom_id na tabela orders e função down
  ```php
  Schema::table('orders',function(Blueprint $table) {
		$table->dropForeign('orders_cupom_id_foreign');
		$table->dropColumn('cupom_id');
	});
  ```
3. Criar Seeder, Factory e adicionar fillables
4. CupomsController
5. Rotas e views para admin.cupoms
6. Refatorar _form e fazer AdminCupomRequest
7. CheckoutController com depencias de OrderRepository, UserRepository e ProductRepository
8. Criação novas rotas e views a partir de 'customer' (no lugar de admin)
9. Javascript para adicionar novos produtos na tela de pedidos 
10. Criação do Service para as orders. Conceitos importantes: 
  - \DB::beginTransaction()
  - try/catch 
  - \DB::commit (dentro do try)
  - DB::rollback() (dentro to catch)	
  - OrderService com dependências de OrderRepository, CupomRepository e ProductRepository
11. Rotas e funções para customer.index e customer.create
12. Permissões de usuários (alteração no middleware checkrole)

###Capítulo 7: OAuth 2

1. Instalando package https://github.com/lucadegasperi/oauth2-server-laravel/wiki
2. Corrigir CSRF (middleware -> kernel.php)
  - Configurar dentro do VerifyCsrfToken para ignorar as rotas da API
  ```php
   protected $except = [
        'oauth/access_token',
		'api/*'
    ];
  ```
3. Authorization Server
  - https://github.com/lucadegasperi/oauth2-server-laravel/wiki/Choosing-a-Grant
  - Escolhido: https://github.com/lucadegasperi/oauth2-server-laravel/wiki/Implementing-an-Authorization-Server-with-the-Password-Grant 
  ```php
  'password' => [
		'class' => '\League\OAuth2\Server\Grant\PasswordGrant',
		'callback' => '\CodeDelivery\OAuth2\PasswordVerifier@verify',
		'access_token_ttl' => 3600
	]
  ```
  - Criar client se for testar requisição do token
4. Refresh token
  - https://github.com/lucadegasperi/oauth2-server-laravel/wiki/Implementing-an-Authorization-Server-with-the-Refresh-Token-Grant
5. Criando rotas para api
  - Route::group(['prefix' => 'api', 'middleware' => 'oauth' , 'as' => 'api.'], function(){ 	//  });
  
###Capítulo 8: Criando API de Cliente

1. Separação das rotas para os diferentes papeis de consumidores da API (client e deliveryman no caso)
  ```php
  Route::group(['prefix' => 'api', 'middleware' => 'oauth' , 'as' => 'api.'], function() 
  {
      Route::group(['prefix' => 'client', 'as' => 'client.'], function() {
      });
      Route::group(['prefix' => 'deliveryman', 'as' => 'deliveryman.'], function() {
      });
  });
  ```
2. Criando middleware oauth.checkrole
  - Inserir no kernel em $routeMiddleware e aplicar nas rotas criadas anteriormente
	```php
	// CodeDelivery\Http\Middleware\OAuthCheckRole 
	public function handle($request, Closure $next, $role)
	{
		$id = Authorizer::getResourceOwnerId();
		$user = $this->userRepository->find($id);

		if ($user->role != $role){
			abort(403,'Acess Forbidden');
		}
		return $next($request);
	}
	```
3. Criando RESTful Controller e rotas
  - ClientCheckoutController
  ```php
  //routes.php
  Route::resource('order', 
  	'Api\Client\ClientCheckoutController',
  	['except' => ['create','edit']]
  );
  ```
4. Criando funcionalidades
  - listagem de orders (index)
  - cadastro de order (store)
  - mostrar uma order (show)
5. Atenção especial:
  - nas funções da API a validação do usuário deve ser feita através do Authorizer
  ```php
  $id = Authorizer::getResourceOwnerId();
        $clientId = $this->userRepository->find($id)->client->id;
  ```
  
###Capítulo 9: Criando API de Deliveryman

1.Criando RESTful Controller e rotas
 - DeliverymanCheckoutController
```php
// routes.php
Route::resource('order',
  'Api\Deliveryman\DeliverymanCheckoutController',
  ['except' => ['create','edit','destroy','store']]
);
```
2.Criando funcionalidades do DeliverymanCheckoutController
  - listagem de orders (index)
  - mostrar uma order (show)
  - atualizar status da order (patch)
    - Atenção: dados enviados através de form-data e x-www-form-urlencoded são interpretados de maneira diferente pelo Request,no caso, utilizamos o x-www-form-urlencoded para que o parametro 'status' fosse lido e atualizado corretamente
3. Nova função adicionada no OrderService para atualizar o status
   ```php
   // OrderService.php
  public function updateStatus(Request $request, $id) 
  {
      $idDeliveryman = Authorizer::getResourceOwnerId();
      $order = $this->orderService->updateStatus($id,$idDeliveryman, $request->get('status'));
      if ($order){
          return $order;
      }
      abort(400,'Order não encontrada'); 
  }
  ```

###Capítulo 10: Validações e Serializações

1. Utilizar camada de validação (Requests) nos controllers da API 
  - Inserir Accept = application/json no HEADER da requisição para definir se a requisição é ajax ou json
2. CheckoutRequest criado
    - Validação de cupoms
    - Validação de items
      - Função para criação de rules dos items
      ```php
      public function buildRulesItems($key, array &$rules)
      {
          $rules["items.$key.product_id"] = 'required';
          $rules["items.$key.qtd"] = 'required';
      }
      ```
    - Atenção: Utilizar 'Illuminate\Http\Request as HttpRequest' no parâmetro da função rules para evitar conflito 
    com o Request que o CheckoutRequest herda
3. Aplicar o mesmo CheckoutRequest para validar o CheckoutController também (store)
4. Serializações
  - Fractal  - PHPLeague (http://fractal.thephpleague.com/)
  - https://github.com/andersao/l5-repository#presenters
  - Para utilizar:
    - Criar Transformer
    ```php
      php artisan make:transformer Order
    ```
    - Criar Presenter
    ```php
      php artisan make:presenter Order
    ```
    - Indicar no repositorio qual é o presenter usado
    ```php
    public function presenter()
    {
        return \CodeDelivery\Presenters\OrderPresenter::class;
    }
    ```
    - Não esquecer do 'use TransformableTrait;'
    - Atenção: para ignorar o presenter e continuar retornando o objeto Eloquent nas consultas deve-se utilizar
    o skipPresenter(). Ex:
    ```php
     $order = $this->orderRepository->skipPresenter()->with(['client','items.product','cupom'])->find($id);
    ```
5. Serialização de relacionamentos
  - protected $defaultIncludes = [''];  Includes que serão automaticamente adicionados
    - Criar funções include juntamente com o transformer do model relacionado. Ex:
    ```php
    public function includeCupom(Order $model)
    {
        if (!$model->cupom)
            return null;
        return $this->item($model->cupom, new CupomTransformer());
    }
    ```
  - protected $availablesIncludes = []; Includes que serão adicionados caso houver requisição
    - Enviar 'include'  no parâmetro da url da requisição com nomes dos includes que quiser adicionar (separados por virgula, sem espaços)
6. Escapando o presenter por padrão (para continuar resgatando o Eloquent normalmente)
  - Para isso é necessário ativar o skipPresenter como padrão no repositório desejado
  ```php
      protected $skipPresenter = true;
  ```
  - Explicitar o presenter nos controllers criados ClientCheckout e DeliverymanCheckout. Ex:
  ```php
      return $this->orderRepository
                  ->skipPresenter(false)
                  ->getByIdAndDeliveryman($id,$idDeliveryman);
  ```   
  
###Capítulo 11: Introdução ao Ionic

1. Componentes
  - Cordova: framework open-source de desenvolvimento mobile que permite utilizar tecnologias web ( HTML5, CSS3 e Javascript ) para criação de aplicações
  multi-plataforma evitando linguagens nativas.
  - AngularJS : framework javascript, foco no modelo, modelo semelhante ao MVC (MVVM),data-binding, suporte a single page application
  - biblioteca ngRoute x Ui-router : ngRoute aceita apenas uma camada de roteamento enquanto o Ui-router é organizado por autômatos finitos
  - NodeJs : biblioteca javascript que funciona no lado do servidor, base das aplicações front-end
  - Bower : gerenciador de dependências para front-end
  - SASS : baseado em Ruby, pré-processador CSS

2. Instalando Ionic e Cordova
  - Pré-requisitos:
    - NodeJS ( https://nodejs.org/ )
      npm é o gerenciador de pacotes do NodeJS
    - Ionic ( http://ionicframework.com/docs/guide/installation.html )
      - MAC : npm install -g cordova ionic
      - Linux : ler Android notes pois é necessário instalar ia32-libs e configurar o java nas variáveis de ambiente antes de instalar o cordova e o ionic
      - Windows :
        1. Instalar JDK7 e configurar variáveis de ambiente corretamente (global)
          - D:\Program Files\apache-ant-1.9.6\bin na PATH
        2. Instalar Apache Ant e configurar variáveis de ambiente corretamente (global)
          - JAVA_HOME ex: D:\Program Files\Java\jdk1.7.0_79
          - %JAVA_HOME%\bin na PATH
        3. Android JDK ( http://developer.android.com/sdk/index.html )
          - instalar android sdk API 22 (Android 5.1.1)
          - ANDROID_HOME ex: D:\Program Files (x86)\Android\android-sdk
          - %ANDROID_HOME%\tools e %ANDROID_HOME%\platform-tools na PATH
        4. 
        ```
         npm install -g cordova ionic
        ```
        
3. Criando primeira aplicação (no caso, ionic é o nome da pasta que será criada)
  ```
    ionic start ionic blank
  ```
  - Estrutura de diretórios
    - package.json : dependências do node, utiliza gulp como gerenciador de tarefas
    - gulpfile.js : configuração das tarefas a serem executadas
    - config.xml : arquivo de configuração do ionic de uso do cordova
    - bower.json : as dependências da nossa aplicação
    - bowerrc : arquivo de configuração do bower, aqui vamos definir o diretorio que serão instalados nossas dependências
    - www : aqui fica nossa aplicação
      - lib : análoga a pasta vendor (não modificar)
    - scss : arquivo de configuração para mudar o template do ionic
    - plugins : todos os plugins do cordova
    - hooks : tarefas que podem ser definidas para o fluxo de trabalho

4. Gerando builds
  - Adicionar plataforma ( para ios é necessário estar utilizando mac )
  
    ```
      $ ionic platform add ios
      $ ionic platform add android
    ```
  - Gerar build
  
    ```
      $ ionic build
    ```
5. Ionic ( http://ionicframework.com/docs/ )
  - Formas de inserir conteúdo:
    1. Diretivas javascript (ex: ion-header-bar, ion-content, ion-footer-bar)
    2. HTML puro
  - Classe importante has-subheader : previne que o conteúdo fique embaixo do subheader
  - Ícones ( http://ionicons.com/ )
    1. Adicionar a classe do ícone a tag desejada
    2. Adicionar tag <i class="icon ion-etc"></i>
  - Muito importante fuçar na documentação!
  
6. Simulação das plataformas
  ```
    ionic serve --labs
  ```

7. Emulação das plataformas
  ```
    ionic emulate "plataforma"
  ```
  
###Capítulo 12: Iniciando com Rotas e Navegação (ui-router)

1. ui-router , não será utilizado o ngRoute do angular ( https://github.com/angular-ui/ui-router )
  - Os Provedores criam serviços que precisamos usar na aplicação, então quando precisamos fazer uma configuração inicial nesses 
  serviços temos que ir no provider, para isso utilizamos o seguinte código no arquivo www/js/app.js
  ```
    .config(function($stateProvider){
        $stateProvider.state('home',{
            url: '/home',
            templateUrl: 'templates/home.html'
        })
    });
  ```
  - Para definir outros estados basta concatenar outro .state() com os paramêtros desejados
  - Criar www/templates para armazenar nossas views
  - Para separar o conteúdo das views e do index.html utilizamos o seguinte código nos templates: (não renderizar uma coisa emcima da outra)
  ```
    <ion-view>
        <ion-content class="padding">
            Minha home
        </ion-content>
    </ion-view>
  ```

2. Navegação entre rotas
  - Primeira maneira: <a href="#/"> Ir </a> (não recomendado)
  - Segunda maneira: Diretiva angular ng-href <a ng-href="#/"> Ir </a>
  - Terceira maneira: Trabalhar com estado ui-sref <a ui-sref="home"> Ir </a> (melhor opção)
    - Para passar valores aos parâmetros diretamente
      ```
        ui-sref="home.b({a: 'fooVal', b: 'barVal'})"
      ```
 
3. Estados filhos (subrotas)
   - Para criar um estado filho basta apenas criar outro estado e concatenar com . o nome do estado pai com o do filho
   ```
     .state('home.a',{
         url: '/a',
         templateUrl: 'templates/home-a.html',
         controller: 'HomeController'
     })
   ```
   - Inserir <ion-nav-view></ion-nav-view> na view do pai no local que se deseja exibir o estado filho

4. Redirecionamento padrão
  - Utilizar o provider do ngRoute ( $urlRouterProvider ) nas nossas configurações e definir a seguinte configuração
  ```
  .config(function($stateProvider, $urlRouterProvider){
      $urlRouterProvider.otherwise('/');
      ...
  ```

5. $state e %stateParams
  - Parâmetro de rota: basta adicionar :nomedoparametro a url das rotas
  - $stateParams é um serviço utilizado para pegar os parâmetros de rotas
  - $state é um serviço que serve para gerenciar o redirecionamento para outros estados via js e também para pegar informações 
  do estado atual
  
6. Controllers
  - Definir o nome do 'controller' dentro do .state() ou definir a função diretamente
  - Para criar um controller é preciso criar um angular.module (neste momento foi criado a pasta js/controllers para organizar os controllers)
  ```
    angular.module('starter.controllers',[])
        .controller('HomeController',
        function($scope, $state, $stateParams){
            $scope.state = $state.current;
            $scope.nome = $stateParams.nome;
        });
  ```
  - Depois de criado, basta importar o js no index.html e adicioná-lo no array de modulos do app.js
  ```
    index.html
      <script src="js/controllers/home.js"></script>
    ...
    app.js
      angular.module('starter', ['ionic', 'starter.controllers'])
    ...
  ```

7. Título das páginas
  ```
    <ion-nav-title>
      Titulo da página que irá aparecer no ion-nav-bar
    </ion-nav-title>
  ```
  
###Capítulo 13: Autenticação e CORS

1. OAuth2 com angular ( https://github.com/seegno/angular-oauth2 )
  - Esta biblioteca utiliza o pacote angular-cookies para gerenciar os tokens
  - A bilbioteca query-string serve para ajudar na formação de urls com GET
  - É obrigatório o uso de HTTPS para manter a segurança, pois o token vai no HEADER da requisição e este deve ser criptografado

2. Instalação da biblioteca
  1. Bower : (lembrar do --save para adicionar a dependencia ao bower.json)
  ```
    bower install angular-oauth2 --save 
  ```
  2. Se aparecer alguma pergunta sobre qual versão instalar, dar preferência àquela que é requerida pelo ionic
  3. Fazer includes no index.html ( a ordem é importante, angular-cookier e query-string precisam vir antes do angular-oauth2)
  ```
    <script src="lib/angular-cookies/angular-cookies.min.js"></script>
    <script src="lib/query-string/query-string.min.js"></script>
    <script src="lib/angular-oauth2/dist/angular-oauth2.min.js"></script>
  ```
  4. Importar o modulo 'angular-oauth2' no app.js
  5. Se a aplicação abrir e não apresentar erros no browser a instalação foi concluída com sucesso

3. Configurações
  1. Chamar o OAuthProvider para iniciarmos as configurações ( arquivo app.js )
     - É este provider que irá gerar o serviço de OAuth e definir suas configurações
    ```
    .config(function($stateProvider, $urlRouterProvider, OAuthProvider){
      OAuthProvider.configure({
          baseUrl: 'http://localhost:8000',
          clientId: 'appid01',
          clientSecret: 'secret' // optional,
          grantPath: '/oauth/access_token'
      });
      ...
    ```
  2. Chamar o OAuthTokenProvider da mesma maneira
    - O OAuthTokenProvider vai gerar o serviço OAuthProvider que vai armazenar o token de acesso nos cookies, o refresh token,
    o nome do cookie, etc
    ```
      OAuthTokenProvider.configure({
        name: 'token',    /* nome do cookie que será criado */
        options: {        /* opções com relação ao cookie que será criado, como tempo de vida, etc */
          secure : false   /* para usar https */ 
          }
      });
    ```
4. Rota e template de login
  1. As rotas do capítulo anterior foram deletadas e uma nova rota para login foi criada, assim como um LoginCtrl
    - templates/login.html
    - js/controllers/login.js
  2. Criação do formulário no login.html
  3. Criação da função login no controller js/login.js
  
    ```
      .controller('LoginCtrl',['$scope', 'OAuth', function($scope, OAuth){ // método explícito 
            // Certifica que o user está vazio
            $scope.user = {
                username: '',
                password: ''
            };
            $scope.login = function (){
                OAuth.getAccessToken($scope.user)
                    .then( function(data){
                        console.log('login funcionando');
                    }, function(responseError) {
                    });
            };
        }]);
    ```

5. CORS ( Cross Origin Resource Sharing )
  - Existe uma configuração que dita o que e a quem será compartilhado informações do servidor, essa configuração é chamada CORS
    - http://www.w3.org/TR/2010/WD-cors-20100727/
  - No caso, instalaremos o pacote https://github.com/barryvdh/laravel-cors para utilizar o cors com o laravel
    - Utilizaremos o cors como um middleware e este será aplicado apenas às rotas da api e do token
    - Para realizar alterações nas configurações é preciso publicar as configurações do pacote, será gerado o arquivo config/cors.php 
    ```
      php artisan vendor:publish --provider="Barryvdh\Cors\ServiceProvider"
    ```
  - Ao aplicar o middleware, a requisição do app mobile para nosso servidor será aceita e o token será gerado. Para acessar o valor do
  token gravado no cookie, existe o serviço '$cookies'
  ```
    angular.module('starter.controllers',[])
    .controller('LoginCtrl',['$scope', 'OAuth', '$cookies', function($scope, OAuth, $cookies){
            $scope.login = function (){
                OAuth.getAccessToken($scope.user)
                    .then( function(data){
                        console.log(data);
                        console.log($cookies.getObject('token')); // 'token' definido no OAuthTokenProvider
                    }, function(responseError) {
                        console.debug(responseError);
                    });
            };
    }]);
  ```

6. Ajustando login e redirecionamentos
  - Adicionar required="required" aos inputs do form
  - Incluir serviço $ionicPopup no controller para exibir popup quando falhar o login
  - Criar home e redirecionar login para a rota criada utilizando o service $state 
  ```
  $scope.login = function (){
    OAuth.getAccessToken($scope.user)
        .then( function(data){
            $state.go('home');
        }, function(responseError) {
            $ionicPopup.alert({
                title: 'Advertência',
                template: 'Login e/ou senha inválidos'
            })
            console.debug(responseError);
        });
  };
  ```