# Laravel Cheatsheet

## Create a Laravel 11 project
```console
composer create-project laravel/laravel [folder-name] "^11"
```

## PHP Artisan Commands

Start local server:
```console
php artisan serve
```


MAKE artisan comands:
```console
# Create with Factory, Migration and Controller
php artisan make:model [model_name] -fmc

php artisan make:migration [migration_file_name]

php artisan make:controller [controller_name] -m [model_name]

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

## Routes

```php

// Using wildcards and named routes
Route::get('/books/{id}', function ($id) {
    return view('jobs');
})->name('books.get');

// Calling named route from HTML
{{ route('books', $id) }}

// Route model binding
Route::get('/books/{book}/edit', function (Book $book) {
    return view('books.edit', ['book' => $book]);
});

// Pass data using controller
Route::get('/books', [JobController::class, 'index']);
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
// Separate defnition of column and setting of FK
$table->unsignedBigInteger('user_id');
$table->foreign('user_id')->references('id')->on('users');

// Definition and setting of FK in one line
$table->foreignId('user_id')->constrained();

// Setting of onUpdate and onCascade options
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
protected $fillable = [
    'title',
    'year',
    'purchase_link'
];

public $table = 'book_listing';
```

Eloquent model database operations:
```php
use App\Models\Book;

// Get all
$books = Book::all();

// Get all with eager loading of "author" relationship
$books = Book::with('author')->get();

// Get book with id equals $book_id
$book = Book::find($book_id);

// Create
$book = Book::create([
    'title' => $title,
    'year' => $year,
    'purchase_link' => $purchaseLink,
]);

// Update
$book->year = 2000;
$book->save();

// Delete
$book->delete();
```

Pagination:
```php
use App\Models\Book;

// Get all books then paginate:
$books = Book::with('author')->paginate(10);
$books = Book::with('author')->simplePaginate(10);
$books = Book::with('author')->cursorPaginate(10);

// Show page links in blade:
{{ $books->links() }}
```

Eloquent model relationships:
```php
use App\Models\Author;
use App\Models\Book;
use App\Models\Genre;
use App\Models\User;

// In Book model:
public function author() {
    return $this->belongsTo(Author::class);
}
public function genres() {
    return $this->belongsToMany(Genre::class);
}

// In Author model:
public function books() {
    return $this->hasMany(Book::class);
}
public function user() {
    return $this->belongsTo(User::class);
}
```

Prevent lazy loading globally:
```php
// In app/Providers/AppServiceProvider.php:

use Illuminate\Database\Eloquent\Model;

Model::preventLazyLoading();
```

## Laravel Blade

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

<!-- Using a named slot --> 
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


## Array Class

```php
use Illuminate\Support\Arr;

// Get first item that matches
Arr::first($jobs, function ($job) use ($id) {
    return $job['id'] == $id;
});

// Using Arrow function
Arr::first($jobs, fn($job) => $job['id'] == $id);
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

To manually throw a validation error:
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

// Use gate in controller method:
Gate::authorize('edit-book', $book);
```

Using Gate + Middleware:
```php
// Define gate in app/Providers/AppServiceProvider.php:
Gate::define('edit-book', function (User $user, Book $book) {
    return $book->author->user->is($user);
});

// In route file:
Route::get('/books/{book}/edit', [BookController::class, 'edit'])
    ->middleware(['auth'])
    ->can('edit-book', 'book');

// In blade file:
@can('book-edit', $book)
@endcan
```

Using Policy:
```php
use App\Http\Controllers\BookController;
use Illuminate\Support\Facades\Route;

// In route file:
Route::get('/books/{book}/edit', [BookController::class, 'edit'])
    ->middleware(['auth'])
    ->can('edit', 'book');

// In blade file:
@can('edit', $book)
@endcan
```
