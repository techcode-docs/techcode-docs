Implementing CRUD operations in Laravel without using PDO but relying on Eloquent ORM (Laravel’s default ORM) is the standard approach. This guide will walk you through the steps to set up Laravel on a local server, implement CRUD operations, and build a UI for it. This tutorial assumes you are working with a local development environment like **XAMPP** or **Laravel Valet**. We'll cover setting up your Laravel project, building the CRUD operations, and creating a UI.

### Step 1: Local Server Installation

#### 1. Install XAMPP (if not already installed)
- Download and install XAMPP from [https://www.apachefriends.org/download.html](https://www.apachefriends.org/download.html).
- Start Apache and MySQL in the XAMPP Control Panel.

#### 2. Install Composer (PHP dependency manager)
- Download and install Composer from [https://getcomposer.org/](https://getcomposer.org/).
- Ensure that Composer is available globally by running `composer --version` in your terminal.

#### 3. Install Laravel
- Open your terminal and navigate to your XAMPP `htdocs` folder:

bash
cd C:\xampp\htdocs


- Create a new Laravel project:

bash
# composer create-project --prefer-dist laravel/laravel crud-laravel


- Navigate into the project directory:

bash
cd crud-laravel


#### 4. Set up the `.env` file for the database
- Open the `.env` file in the Laravel project folder and configure the database connection:

env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=crud_db
DB_USERNAME=root
DB_PASSWORD=


#### 5. Create the MySQL Database
- Go to `http://localhost/phpmyadmin` and create a new database named `crud_db`.

### Step 2: File Directory Structure

After setting up Laravel, your directory structure will look like this:


/crud-laravel
   ├── /app
   │   └── /Http/Controllers
   │        └── UserController.php
  

 ├── /database
   │   └── /migrations
   │        └── 2024_09_17_create_users_table.php
   ├── /resources
   │   └── /views
   │        └── /users
   │            └── index.blade.php
   │            └── create.blade.php
   │            └── edit.blade.php
   │        └── layout.blade.php
   └── /routes
       └── web.php


We will be working mostly with these folders:

- `/app/Http/Controllers`: For CRUD controller files.
- `/resources/views`: For frontend UI.
- `/routes/web.php`: For defining routes.
- `/database/migrations`: For creating database tables.

### Step 3: Implement CRUD Operations

#### 1. Create a Model and Migration for Users
- Run the following command to generate the `User` model and migration:

bash
# php artisan make:model User -m


- Open the migration file created under `/database/migrations/` and modify it to add the fields for the `users` table:

<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateUsersTable extends Migration
{
    public function up()
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email');
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('users');
    }
}


- Run the migration to create the `users` table:

bash
# php artisan migrate


#### 2. Create the CRUD Controller
- Run the following command to generate the controller:

bash
# php artisan make:controller UserController


- Open the newly created controller in `/app/Http/Controllers/UserController.php` and add the following code for the CRUD operations:

<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\User;

class UserController extends Controller
{
    // Display the users list (Read)
    public function index()
    {
        $users = User::all();
        return view('users.index', compact('users'));
    }

    // Show the form for creating a new user (Create)
    public function create()
    {
        return view('users.create');
    }

    // Store a new user in the database (Create)
    public function store(Request $request)
    {
        $request->validate([
            'name' => 'required',
            'email' => 'required|email'
        ]);

        User::create($request->all());

        return redirect()->route('users.index')->with('success', 'User created successfully.');
    }

    // Show the form for editing the specified user (Update)
    public function edit(User $user)
    {
        return view('users.edit', compact('user'));
    }

    // Update the specified user in the database (Update)
    public function update(Request $request, User $user)
    {
        $request->validate([
            'name' => 'required',
            'email' => 'required|email'
        ]);

        $user->update($request->all());

        return redirect()->route('users.index')->with('success', 'User updated successfully.');
    }

    // Remove the specified user from the database (Delete)
    public function destroy(User $user)
    {
        $user->delete();

        return redirect()->route('users.index')->with('success', 'User deleted successfully.');
    }
}


#### 3. Define Routes for CRUD
- Open `/routes/web.php` and define the routes for the CRUD operations:

<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\UserController;

Route::get('/', [UserController::class, 'index']);
Route::resource('users', UserController::class);


#### 4. Create Views for CRUD Operations
- Under the `/resources/views` folder, create a `users` directory and add the following views:


##### `index.blade.php`: Display all users
@extends('layout')

@section('content')
<h2>Users List</h2>
<a href="{{ route('users.create') }}">Create New User</a>

@if(session('success'))
    <p>{{ session('success') }}</p>
@endif

<table border="1" cellpadding="10">
    <tr>
        <th>ID</th>
        <th>Name</th>
        <th>Email</th>
        <th>Actions</th>
    </tr>
    @foreach ($users as $user)
    <tr>
        <td>{{ $user->id }}</td>
        <td>{{ $user->name }}</td>
        <td>{{ $user->email }}</td>
        <td>
            <a href="{{ route('users.edit', $user->id) }}">Edit</a>
            <form action="{{ route('users.destroy', $user->id) }}" method="POST" style="display:inline-block;">
                @csrf
                @method('DELETE')
                <button type="submit">Delete</button>
            </form>
        </td>
    </tr>
    @endforeach
</table>
@endsection




##### `create.blade.php`: Create new user
@extends('layout')

@section('content')
<h2>Create New User</h2>

@if($errors->any())
    <ul>
        @foreach($errors->all() as $error)
            <li>{{ $error }}</li>
        @endforeach
    </ul>
@endif

<form action="{{ route('users.store') }}" method="POST">
    @csrf
    Name: <input type="text" name="name" value="{{ old('name') }}"><br>
    Email: <input type="text" name="email" value="{{ old('email') }}"><br>
    <button type="submit">Create</button>
</form>

<a href="{{ route('users.index') }}">Back</a>
@endsection




##### `edit.blade.php`: Edit existing user
@extends('layout')

@section('content')
<h2>Edit User</h2>

@if($errors->any())
    <ul>
        @foreach($errors->all() as $error)
            <li>{{ $error }}</li>
        @endforeach
    </ul>
@endif

<form action="{{ route('users.update', $user->id) }}" method="POST">
    @csrf
    @method('PUT')
    Name: <input type="text" name="name" value="{{ $user->name }}"><br>
    Email: <input type="text" name="email" value="{{ $user->email }}"><br>
    <button type="submit">Update</button>
</form>

<a href="{{ route('users.index') }}">Back</a>
@endsection




##### `layout.blade.php`: Layout Template
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Laravel CRUD</title>
</head>
<body>

    <div class="container">
        @yield('content')
    </div>

</body>
</html>


### Step 4: Running the Application

1. Start your Laravel local development server:

bash
php artisan serve


2. Open your browser and go to: `http://localhost:8000/users` to see the list of users.

