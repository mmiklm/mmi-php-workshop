# Laravel Cheatsheet

<!---
your comment goes here
and here
-->

## Create a Laravel 11 project
```console
composer create-project laravel/laravel [project-folder-name] "^11"
```

## PHP Artisan Commands

View all artisan commands:
```console
php artisan
```

View options for specific artisan command:
```console
# e.g. php artisan help db:seed
php artisan help [command]
```

Start local server:
```console
php artisan serve
```


MAKE artisan comands:
```console
# Create with Factory, Migration and Controller
php artisan make:model [model_name] -fmc

# e.g. php artisan make:migration create_books_table
php artisan make:migration [migration_file_name]

# e.g. php artisan make:controller BookController -m Book
php artisan make:controller [controller_name] -m [model_name]

# e.g. php artisan make:controller BookFactory -m Book
php artisan make:factory [factory_name] -m [model_name]

php artisan make:seeder [seeder_name]
```


Execute/rollback migration:

```console
# Run new migration files
php artisan migrate

# Drop all tables and rerun migration
php artisan migrate:fresh

# Drop all tables, rerun migration and run seeder
php artisan migrate:fresh --seed

# Rollback all migrations and rerun migration
php artisan migrate:refresh

# Rollback last migration
php artisan migrate:rollback
```

Run database seeder:

```console
# Run default seeder class "Database\Seeders\DatabaseSeeder"
php artisan db:seed

# Run specific seeder class (e.g. BookSeeder)
php artisan db:seed --class=BookSeeder
```

## Routes

```php
// Use wildcards and named routes
// e.g. Handle call to /books/1
Route::get('/books/{id}', function ($id) {
    return view('books');
})->name('books.get');

// Call named route from HTML
// e.g. Call /books/1
{{ route('books', 1) }} 

// Route model binding
// e.g. Handle call to /books/1/edit
Route::get('/books/{book}/edit', function (Book $book) {
    return view('books.edit', ['book' => $book]);
});

// Use controller
Route::get('/books', [BookController::class, 'index']);
```


## Migration

```php
// Create table
Schema::create('books', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->text('description')->nullable();
    $table->string('author', 100);
    $table->integer('year');
    $table->timestamps();
});

// Update table
Schema::table('books', function (Blueprint $table) {
    $table->string('purchase_link')->after('year');
});

// Drop table
Schema::dropIfExists('books');

// Drop column
Schema::table('books', function (Blueprint $table) {
    $table->dropColumn('description');
});
```

Set foreign keys:
```php
// Define column and set FK separately
$table->unsignedBigInteger('user_id');
$table->foreign('user_id')->references('id')->on('users');

// Define column and set FK in one line
$table->foreignId('user_id')->constrained();

// Set onUpdate and onCascade options
$table->foreignId('user_id')
    ->constrained()
    ->onUpdate('cascade')
    ->onDelete('cascade');

// If column name is non-standard format 
// (e.g. uses books_user_id instead of user_id)
$table->foreignId('user_id')->constrained(
    table: 'users', indexName: 'books_user_id'
);

// Use column "modifiers" before call to constrained()
$table->foreignId('user_id')
    ->nullable()
    ->constrained();

// Use model class
$table->foreignIdFor(User::class);
```

Drop foreign key:
```php
$table->dropForeign('books_user_id_foreign');
$table->dropIndex('books_user_id_foreign');
```


## Eloquent Model

Eloquent model overrides:

```php
// Allow mass assignment for certain fields
protected $fillable = [
    'title',
    'year',
    'purchase_link'
];

// Allow mass assignment for all fields except
protected $guard = [
    'id'
];

// Do not return
protected $hidden = [
    'password',
    'remember_token',
];

// Define if name of table does not match model name
// e.g. "book_listing" instead of "books"
public $table = 'book_listing';

// Do not use timestamps when creating table
// Need to remove $table->timestamps() in migration
public $timestamps = false;

// Automatically cast data in column when fetched
protected function casts(): array
{
    return [
        'created_at' => 'datetime:Y-m-d',
    ];
}

```

Eloquent model database operations:

```php
use App\Models\Book;

// Get all
$books = Book::all();

// Get all with eager loading of "author" relationship
$books = Book::with('author')->get();

// Get book with id equals $bookId
$book = Book::find($bookId);

// Get all books where year > 2000
$book = Book::where('year', '>', 2000)->get();

// Create
$book = Book::create([
    'title' => $title,
    'year' => $year,
    'purchase_link' => $purchaseLink,
]);

// Update or Create
$book = Book::updateOrCreate(
    [
        'title' => $title
    ],
    [
        'description' => $description,
        'year' => $year,
    ]
);

// Retrieve and update
$book = Book::find(1);
$book->year = 2000;
$book->save();

// Retrieve and delete
$book = Book::find(1);
$book->delete();

// Delete by ID
Book::destroy(1);
```

Pagination:
```php
use App\Models\Book;

// Get all books then paginate
$books = Book::with('author')->paginate(10);
$books = Book::with('author')->simplePaginate(10);
$books = Book::with('author')->cursorPaginate(10);

// Show page links in blade
{{ $books->links() }}
```

Eloquent model relationships:
```php
use App\Models\Author;
use App\Models\Book;
use App\Models\Genre;
use App\Models\User;

// In Book model
public function author() {
    return $this->belongsTo(Author::class);
}
public function genres() {
    return $this->belongsToMany(Genre::class);
}

// In Author model
public function books() {
    return $this->hasMany(Book::class);
}
public function user() {
    return $this->belongsTo(User::class);
}
```

Insert/Delete record in a many-to-many relationship's intermediate table:
```php
$author = Author::find(1);

// Attach a single book to the author
$author->books()->attach($bookId);

// Detach all books then attach defined books
$user->books()->sync([1, 2, 3]);

// Attach multiple books without detaching
$user->roles()->syncWithoutDetaching([1, 2, 3]);

// Detach a single book from the author
$author->books()->detach($bookId);
$user->books()->detach([1, 2, 3]);

// Detach all books from the author
$author->books()->detach();
```

Prevent lazy loading globally:
```php
// In app/Providers/AppServiceProvider.php:

use Illuminate\Database\Eloquent\Model;

Model::preventLazyLoading();
```

## Blade Templates

Using Components:

```html
<!-- Create component --> 
<a {{ $attributes }}>
    {{ $slot }} 
</a>

<!-- Use component --> 
<x-nav-link href="/">Home</x-nav-link>

<!-- Merge $attributes with 'class' attribute -->
<a {{ $attributes->merge(['class' => 'bg-blue-500 hover:bg-blue-600 text-white font-bold p-4 rounded']) }} 
>
    {{ $slot }}
</a>

<!-- Use a named slot --> 
<x-slot:heading>Home Page</x-slot:heading>

<!-- Refer to named slot --> 
<div>{{ $heading }}</div>
```

Directives:
```php
@props(['active' => false])

@foreach ( as )
@endforeach

@if()
@endif

@csrf

@method('DELETE')

@error('email')
    {{ $message }}
@enderror
```

## Collection

See all available collection methods in the official Laravel documentation: https://laravel.com/docs/11.x/collections#available-methods

```php
// Create collection
$collection = collect([1, 2, 3, 4, 5]);

// Get element from collection using key
$collection = collect([1, 2, 3, 4, 5]);
$element = $collection->get(2); // 3

// Convert collection to array
$arrayFromCollection = $collection->toArray();

// "Pluck" value from collection
$collection = collect([
    ['product_id' => 'prod-100', 'name' => 'Desk'],
    ['product_id' => 'prod-200', 'name' => 'Chair'],
]);
$plucked = $collection->pluck('name');
$plucked->all(); // ['Desk', 'Chair']

// "Pluck" value from collection and specify key
$collection = collect([
    ['product_id' => 'prod-100', 'name' => 'Desk'],
    ['product_id' => 'prod-200', 'name' => 'Chair'],
]);
$plucked = $collection->pluck('name', 'product_id');
$plucked->all();
// ['prod-100' => 'Desk', 'prod-200' => 'Chair']

// Use filter() method
$collection = collect([1, 2, 3, 4, 5]);
$filtered = $collection->filter(function (int $value, int $key) {
    return $value > 2;
});
$filtered->all(); // [3, 4, 5]

// Use map() method
$collection = collect([1, 2, 3, 4, 5]);
$multiplied = $collection->map(function (int $item, int $key) {
    return $item * 2;
});
$multiplied->all(); // [2, 4, 6, 8, 10]



```


## Array Class

```php
use Illuminate\Support\Arr;

// Get first item that matches
Arr::first($books, function ($book) use ($id) {
    return $book['id'] == $id;
});

// Use arrow function
Arr::first($books, fn($book) => $book['id'] == $id);
```


## Auth Class
```php
use Illuminate\Support\Facades\Auth;

Auth::user() // get user
Auth::id() // get user's ID
Auth::guest() // is guest
Auth::auth() // is authenticated
Auth::login($user) // log in
Auth::logout() // log out

// attempt login
Auth::attempt([ 
    'email' => $email,
    'password' => $password,
]) 
```


## Forms

Input validation:
```php
use Illuminate\Http\Request;
use Illuminate\Validation\Rules\Password;

$request->validate([
    'name' => ['required', 'max:255'],
    'email' => ['required', 'email', 'max:255'],
    'password' => ['required', Password::min(8), 'confirmed'],
]);
```

See all available validation rules in the official Laravel documentation: https://laravel.com/docs/11.x/validation#available-validation-rules


Manually throw a validation error:
```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Validation\ValidationException;

$credentials = [
    'email' => $request->email,
    'password' => $request->password,
];

if (!Auth::attempt($credentials)) {
    throw ValidationException::withMessages([
        'password' => 'Sorry, those credentials do not match.'
    ]);
}
```

Regenerate session:
```php
request()->session()->regenerate();
```

Check if current request matches the pattern:
```php
request()->is('/pattern');
```

## Resource Authorization
Using manual checking:
```php
// In controller method:
if ($book->author->user->isNot(Auth::user())) {
    abort(403);
}
```

Using Gate:
```php
// Define gate in app/Providers/AppServiceProvider.php:
Gate::define('edit-book', function (User $user, Book $book) {
    return $book->author->user->is($user);
});

// Use gate in controller method
Gate::authorize('edit-book', $book);
```

Using Gate + Middleware:
```php
// Define gate in app/Providers/AppServiceProvider.php:
Gate::define('edit-book', function (User $user, Book $book) {
    return $book->author->user->is($user);
});

// In route file
Route::get('/books/{book}/edit', [BookController::class, 'edit'])
    ->middleware(['auth'])
    ->can('edit-book', 'book');

// In blade file
@can('book-edit', $book)
@endcan
```

Using Policy:
```php
use App\Http\Controllers\BookController;
use Illuminate\Support\Facades\Route;

// In route file
Route::get('/books/{book}/edit', [BookController::class, 'edit'])
    ->middleware(['auth'])
    ->can('edit', 'book');

// In blade file
@can('edit', $book)
@endcan
```

## Asset Bundling using Vite

```php
# Hot reload asset bundling (for development)
npm run dev

# Bundle assets for deployment
npm run build
```
