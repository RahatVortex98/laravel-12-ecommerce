
<h3 align='center'>🛒🛍️laravel-12-ecommerce</h3>

## Core Features
- **Authentication & Authorization**  
  Multi-role authentication (admin and user) with registration, login, email verification, and password reset.

- **Admin Panel**  
  Full CRUD operations for categories and products (including image uploads and management).  
  Order management with status updates and viewing all orders.

- **Frontend & User Features**  
  Responsive product listing with search, category filtering, and pagination.  
  Detailed product pages with short/long descriptions.  
  Shopping cart (add, update quantity, remove items) with real-time total calculation.  
  Checkout process and order placement.

- **Order & Payment**  
  Conversion of cart items to orders.  
  Order history for users.  
  Invoice generation and PDF download.  
  Stripe payment gateway integration for secure checkout.

- **Other**  
  Dynamic frontend template integration.  
  Secure middleware for role-based access.  
  Basic input validation and error handling.

### Technologies Used
- PHP 8.2+  
- Laravel 12 (with Breeze for authentication scaffolding)  
- MySQL / MariaDB  
- Bootstrap (for responsive UI)  
- Stripe PHP SDK

## Multiple Authentication using Breeze

```cmd
composer require laravel/breeze --dev
```
And then,

```cmd
 php artisan breeze:install
```
- **select -> blade -> dark/light mode/ and pest**

Then Migration,

```cmd
php artisan migrate
```
For breeze (Downloads the design tools):

```cmd
npm install
```
Compiles the design so you can see it
```cmd
npm run dev
```
- **Need to add another column in the user table of user_type**

```cmd
 php artisan make:migration add_column_to_users --table=users
```
In the new migration file:

```cmd
$table->string('user_type')->after('name')->default('user');
```
And then,

```cmd
php artisan migrate
```

<h4> Separate Dashboard</h4>

- **Step 1: Redirect Users After Login**
  
- **By default, Breeze sends everyone to /dashboard. You need to change this in
  
  app/Http/Controllers/Auth/AuthenticatedSessionController.php**

```code
public function store(LoginRequest $request): RedirectResponse
{
    $request->authenticate();
    $request->session()->regenerate();

    // Check user_type and redirect accordingly
    if ($request->user()->user_type === 'admin') {
        return redirect()->intended(route('admin.dashboard'));
    }

    return redirect()->intended(route('dashboard'));
}

```

- **Step 2: Redirect Users After Login**

```cmd
php artisan make:middleware AdminMiddleware
```
Open app/Http/Middleware/AdminMiddleware.php and update the handle function:

```code
public function handle(Request $request, Closure $next): Response
{
    if (auth()->check() && auth()->user()->user_type === 'admin') {
        return $next($request);
    }

    // If not admin, send them back to the user dashboard
    return redirect('dashboard')->with('error', 'You do not have admin access.');
}

```
- **Step 3: Register the Middleware in bootstrap/app.php**

```code
    ->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'admin' => \App\Http\Middleware\AdminMiddleware::class,
    ]);
    })
```
- **Step 4: routes/web.php**

```code
use App\Http\Controllers\ProfileController;
use Illuminate\Support\Facades\Route;

// Normal User Dashboard
Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware(['auth', 'verified'])->name('dashboard');

// Admin Dashboard
Route::get('/admin/dashboard', function () {
    return view('admin_dashboard'); // Create this file in resources/views
})->middleware(['auth', 'verified'])->name('admin.dashboard');
```
- **Step 5: resources/views/admin_dashboard.blade.php**

```code
<x-app-layout>
    <x-slot name="header">
        <h2 class="font-semibold text-xl text-gray-800 leading-tight">
            {{ __('Admin Dashboard') }}
        </h2>
    </x-slot>

    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                <div class="p-6 text-gray-900">
                    {{ __("Welcome, Admin! You have special access.") }}
                </div>
            </div>
        </div>
    </div>
</x-app-layout>


```

<h3 align="center">📊 Dashboard Overview</h3>

<table align="center">
  <tr>
    <th>Admin Dashboard</th>
    <th>User Dashboard</th>
  </tr>
  <tr>
    <td align="center">
      <img
        src="https://github.com/user-attachments/assets/4fcdc957-bed7-49b5-abf5-c56029d8f6fa"
        alt="Admin Dashboard"
        width="600"
      />
    </td>
    <td align="center">
      <img
        src="https://github.com/user-attachments/assets/99db109e-7c2a-49a1-a5f8-a0d7279b28fd"
        alt="User Dashboard"
        width="600"
      />
    </td>
  </tr>
</table>



## Frontend Part and steps:

- **Step 1: Move Assets (CSS, JS, Images, Fonts)**

1. Copy the `css`, `js`, `images`, and `fonts` folders  
2. Paste them into your Laravel project's **public** folder  

        Path: your-project/public/

- **Step 2: Create the "Master Layout"**

1. Go to resources/views/layouts/.

2. Create a new file called site.blade.php.

3. Open the index.html, copy everything, and paste it into site.blade.php.    

- **Step 3: Clean up the Master Layout"**
  
In site.blade.php, find the "middle" content (the part that changes between the Home page and the Shop page) and replace it with:

```PHP
@yield('content')
```
Then, fix the links for the CSS and JS files so Laravel can find them. Change:

    <link href="css/style.css"> → <link href="{{ asset('css/style.css') }}">

    <script src="js/main.js"> → <script src="{{ asset('js/main.js') }}">

    <img src="images/logo.png"> → <img src="{{ asset('images/logo.png') }}">
    
- **Step 4: Clean up the Master Layout"**

Now, you will make your welcome.blade.php (your homepage) use that master layout.

1. Open resources/views/welcome.blade.php.

2. Delete everything inside and replace it with:

```PHP
@extends('layouts.site')

@section('content')
    <h1>Welcome to our Shop</h1>
@endsection

```
Repeat this for shop.blade.php, contact.blade.php, etc.


- **Step 5: Update the Routes"**

```PHP
Route::get('/', function () {
    return view('welcome');
});

Route::get('/shop', function () {
    return view('shop');
});

Route::get('/contact', function () {
    return view('contact');
});

```

## Login, Logout, and Registration:

- **If he is a user and logged in, he will see my dashboard and logout option; if he isn't logged in, he will see register and login**
- **if he is an admin, only see log out and dashboard**

```PHP

    @if (Route::has('login'))
             @auth
        @if(Auth::user()->user_type === 'admin')
            <a href="{{ route('admin.dashboard') }}">
                <i class="fa fa-vcard" aria-hidden="true"></i>
                <span>Admin Panel</span>
            </a>
        @else
            <a href="{{ url('/dashboard') }}">
                <i class="fa fa-user" aria-hidden="true"></i>
                <span>My Dashboard</span>
            </a>
        @endif

        <form method="POST" action="{{ route('logout') }}" style="display: inline;">
            @csrf
            <a href="{{ route('logout') }}" 
               onclick="event.preventDefault(); this.closest('form').submit();">
                <i class="fa fa-sign-out" aria-hidden="true"></i>
                <span>Logout</span>
            </a>
        </form>

    @else
        <a href="{{ route('login') }}">
            <i class="fa fa-sign-in" aria-hidden="true"></i>
            <span>Login</span>
        </a>

        @if (Route::has('register'))
            <a href="{{ route('register') }}">
                <i class="fa fa-user-plus" aria-hidden="true"></i>
                <span>Register</span>
            </a>
        @endif
    @endauth
@endif
```
## User/Admin after logging in, they will see the website first:

- **Step 1: Change the Login Redirect Logic"**

Open the file: app/Http/Controllers/Auth/AuthenticatedSessionController.php

```PHP
    public function store(LoginRequest $request): RedirectResponse
    {
        $request->authenticate();

        $request->session()->regenerate();

        // Check user_type and redirect accordingly
        if ($request->user()->user_type === 'admin') {
            return redirect()->intended('/');
        }

    return redirect()->intended('/');
    }
```
- **Step 2: Change the "HOME" constant**
Open: app/Providers/RouteServiceProvider.php (or app/Http/Requests/Auth/LoginRequest.php in some versions).

Find this line:

```PHP
public const HOME = '/dashboard';

```

Change it to:

```PHP
public const HOME = '/';

```

- **Step 3: Handle the Registration Redirect**

Open: app/Http/Controllers/Auth/RegisteredUserController.php

```PHP
return redirect('/');

```

