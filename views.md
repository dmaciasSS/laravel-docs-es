# Views

- [Uso](#uso)
- [View Composers](#view-composers)

<a name="uso"></a>
## Uso

Vistas contiene el HTML servido por tu aplicación y sirve como un conveniente metodo de separar tu controlador y logica de dominio de to logica de presentación. Vistas se guardan dentro el directorio 'resources/view'.

Una simple vista se ve asi:

	<!-- Vista guardada dentro de resources/views/saludo.php -->

	<html>
		<body>
			<h1>Hola, <?php echo $nombre; ?></h1>
		</body>
	</html>

La vista puede ser regresada ha tu navegador de web de esta forma:

	Route::get('/', function()
	{
		return view('saludo', ['nombre' => 'Jaime']);
	});

Como puedes ver el primer argumento pasado al metodo de ayuda de `view` corresponde al nombre del archivo de la vista en el directorio `resources/views`. El segundo argumento pasado al metodo de ayuda es una matriz de datos que esta disponible ha la vista.

Por supuesto vistas tambien pueden ser encadenadas en un subdirectorio en el directorio `resources/views`. Por ejemplo, si tu vista esta archivada en `resources/views/admin/perfil.php`, debe regresar asi:

	return view('admin.perfil', $data);

#### Pasando Datos A Vistas

	// Usando convención normal
	$view = view('saludo')->with('nombre', 'Victoria');

	// Usando Metodo Magico
	$view = view('saludo')->withNombre('Victoria');

En el ejemplo de arriva, la variable `$nombre` se le da ha aceder a la vista y contineu `Victoria`.
In the example above, the variable `$name` is made accessible to the view and contains `Victoria`.

Si deseas, puedes pasard una matriz de datos como el segundo parametro al metod de ayuda de `view`
If you wish, you may pass an array of data as the second parameter to the `view` helper:

	$view = view('saludo', $data);

#### Sharing Data With All Views

Occasionally, you may need to share a piece of data with all views that are rendered by your application. You have several options: the `view` helper, the `Illuminate\Contracts\View\Factory` [contract](/5.0/contracts), or a wildcard [view composer](#view-composers).

For example, using the `view` helper:

	view()->share('data', [1, 2, 3]);

You may also use the `View` facade:

	View::share('data', [1, 2, 3]);

Typically, you would place calls to the `share` method within a service provider's `boot` method. You are free to add them to the `AppServiceProvider` or generate a separate service provider to house them.

> **Note:** When the `view` helper is called without arguments, it returns an implementation of the `Illuminate\Contracts\View\Factory` contract.

#### Determining If A View Exists

If you need to determine if a view exists, you may use the `exists` method:

	if (view()->exists('emails.customer'))
	{
		//
	}

#### Returning A View From A File Path

If you wish, you may generate a view from a fully-qualified file path:

	return view()->file($pathToFile, $data);

<a name="view-composers"></a>
## View Composers

View composers are callbacks or class methods that are called when a view is rendered. If you have data that you want to be bound to a view each time that view is rendered, a view composer organizes that logic into a single location.

#### Defining A View Composer

Let's organize our view composers within a [service provider](/5.0/providers). We'll use the `View` facade to access the underlying `Illuminate\Contracts\View\Factory` contract implementation:

	<?php namespace App\Providers;

	use View;
	use Illuminate\Support\ServiceProvider;

	class ComposerServiceProvider extends ServiceProvider {

		/**
		 * Register bindings in the container.
		 *
		 * @return void
		 */
		public function boot()
		{
			// Using class based composers...
			View::composer('profile', 'App\Http\ViewComposers\ProfileComposer');

			// Using Closure based composers...
			View::composer('dashboard', function()
			{

			});
		}

		/**
		 * Register
		 *
		 * @return void
		 */
		public function register()
		{
			//
		}

	}

> **Note:** Laravel does not include a default directory for view composers. You are free to organize them however you wish. For example, you could create an `App\Http\ViewComposers` directory.

Remember, you will need to add the service provider to the `providers` array in the `config/app.php` configuration file.

Now that we have registered the composer, the `ProfileComposer@compose` method will be executed each time the `profile` view is being rendered. So, let's define the composer class:

	<?php namespace App\Http\ViewComposers;

	use Illuminate\Contracts\View\View;
	use Illuminate\Users\Repository as UserRepository;

	class ProfileComposer {

		/**
		 * The user repository implementation.
		 *
		 * @var UserRepository
		 */
		protected $users;

		/**
		 * Create a new profile composer.
		 *
		 * @param  UserRepository  $users
		 * @return void
		 */
		public function __construct(UserRepository $users)
		{
			// Dependencies automatically resolved by service container...
			$this->users = $users;
		}

		/**
		 * Bind data to the view.
		 *
		 * @param  View  $view
		 * @return void
		 */
		public function compose(View $view)
		{
			$view->with('count', $this->users->count());
		}

	}

Just before the view is rendered, the composer's `compose` method is called with the `Illuminate\Contracts\View\View` instance. You may use the `with` method to bind data to the view.

> **Note:** All view composers are resolved via the [service container](/5.0/container), so you may type-hint any dependencies you need within a composer's constructor.

#### Wildcard View Composers

The `composer` method accepts the `*` character as a wildcard, so you may attach a composer to all views like so:

	View::composer('*', function()
	{
		//
	});

#### Attaching A Composer To Multiple Views

You may also attach a view composer to multiple views at once:

	View::composer(['profile', 'dashboard'], 'App\Http\ViewComposers\MyViewComposer');

#### Defining Multiple Composers

You may use the `composers` method to register a group of composers at the same time:

	View::composers([
		'App\Http\ViewComposers\AdminComposer' => ['admin.index', 'admin.profile'],
		'App\Http\ViewComposers\UserComposer' => 'user',
		'App\Http\ViewComposers\ProductComposer' => 'product'
	]);

### View Creators

View **creators** work almost exactly like view composers; however, they are fired immediately when the view is instantiated. To register a view creator, use the `creator` method:

	View::creator('profile', 'App\Http\ViewCreators\ProfileCreator');
