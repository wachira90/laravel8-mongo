# install laravel 8 with mongodb CRUD

## composer init project
````
composer create-project --prefer-dist laravel/laravel laravel8_mongo

composer create-project laravel/laravel laravel8_mongo
````

## .env 

````
MONGO_DB_HOST=172.16.1.5
MONGO_DB_PORT=27017
MONGO_DB_DATABASE=testdb
MONGO_DB_USERNAME=root
MONGO_DB_PASSWORD=toor
````


## config/database.php
````
'connections' => [

     'mongodb' => [
            'driver'   => 'mongodb',
            'host'     => env('MONGO_DB_HOST', 'localhost'),
            'port'     => env('MONGO_DB_PORT', 27017),
            'database' => env('MONGO_DB_DATABASE'),
            'username' => env('MONGO_DB_USERNAME'),
            'password' => env('MONGO_DB_PASSWORD'),
            'options'  => []
        ],
        
    ]
````

## install mongo driver
````
composer require jenssegers/mongodb
````

## config/app.php
````
return [
 ....
 'providers' => [
 ....
 Jenssegers\Mongodb\MongodbServiceProvider::class,
 ]
 .....
]
````

## routes/web.php
````
use App\Http\Controllers\StudentController;

Route::resource('student',StudentController::class);
````

## Create a Model and Controller
````
php artisan make:controller StudentController --resource --model=Student
````

## app\Models\Student.php
````
<?php
namespace App\Models;
use Illuminate\Database\Eloquent\Factories\HasFactory;
//use Illuminate\Database\Eloquent\Model;
use Jenssegers\Mongodb\Eloquent\Model;

class Student extends Model
{
    use HasFactory;
    protected $connection = 'mongodb';
    protected $collection = 'students';
    protected $fillable = [
        'first_name','last_name', 'address'
    ];
}
````

## app\Http\Controllers\StudentController.php
````
<?php
namespace App\Http\Controllers;

use App\Models\Student;
use Illuminate\Http\Request;

class StudentController extends Controller
{
    /**
     * Display a listing of the resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function index()
    {
        //
        $students = Student::all();
        return view('student.list', compact('students','students'));
        // return "hello";
    }

    /**
     * Show the form for creating a new resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function create()
    {
        //
        return view('student.create');
    }

    /**
     * Store a newly created resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        //
        $request->validate([
            'txtFirstName'=>'required',
            'txtLastName'=> 'required',
            'txtAddress' => 'required'
        ]);

        $student = new Student([
            'first_name' => $request->get('txtFirstName'),
            'last_name'=> $request->get('txtLastName'),
            'address'=> $request->get('txtAddress')
        ]);

        $student->save();
        
        return redirect('/student')->with('success', 'Student has been added');
    }

    /**
     * Display the specified resource.
     *
     * @param  \App\Student  $student
     * @return \Illuminate\Http\Response
     */
    public function show(Student $student)
    {
        //
        return view('student.view',compact('student'));
    }

    /**
     * Show the form for editing the specified resource.
     *
     * @param  \App\Student  $student
     * @return \Illuminate\Http\Response
     */
    public function edit(Student $student)
    {
        //
        return view('student.edit',compact('student'));
    }

    /**
     * Update the specified resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \App\Student  $student
     * @return \Illuminate\Http\Response
     */
    public function update(Request $request,$id)
    {
        $request->validate([
            'txtFirstName'=>'required',
            'txtLastName'=> 'required',
            'txtAddress' => 'required'
        ]);
        $student = Student::find($id);
        $student->first_name = $request->get('txtFirstName');
        $student->last_name = $request->get('txtLastName');
        $student->address = $request->get('txtAddress');
        $student->update();
        return redirect('/student')->with('success', 'Student updated successfully');
    }

    /**
     * Remove the specified resource from storage.
     *
     * @param  \App\Student  $student
     * @return \Illuminate\Http\Response
     */
    public function destroy(Student $student)
    {
        $student->delete();
        return redirect('/student')->with('success', 'Student deleted successfully');
    }

}
````

## resources\views\student\layouts\app.blade.php
````
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Bootstrap Example</title>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.4.0/css/bootstrap.min.css">
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.4.0/js/bootstrap.min.js"></script>
</head>
<body>
<div class="container">
    @yield('content')
</div>
</body>
</html>
````

## resources\views\student\list.blade.php
````
@extends('student.layouts.app')

@section('content')
    <div class="row">
        <div class="col-lg-11">
                <h2>Laravel 8 Mongodb CRUD Tutorial Example</h2>
        </div>
        <div class="col-lg-1">
            <a class="btn btn-success" href="{{ route('student.create') }}">Add</a>
        </div>
    </div>

    @if ($message = Session::get('success'))
        <div class="alert alert-success">
            <p>{{ $message }}</p>
        </div>
    @endif

    <table class="table table-bordered">
        <tr>
            <th>No</th>
            <th>First Name</th>
            <th>Last Name</th>
            <th>Address</th>
            <th width="280px">Action</th>
        </tr>
        @php
            $i = 0;
        @endphp
        @foreach ($students as $student)
            <tr>
                <td>{{ ++$i }}</td>
                <td>{{ $student->first_name }}</td>
                <td>{{ $student->last_name }}</td>
                <td>{{ $student->address }}</td>
                <td>
                    <form action="{{ route('student.destroy',$student->id) }}" method="POST">
                        <a class="btn btn-info" href="{{ route('student.show',$student->id) }}">Show</a>
                        <a class="btn btn-primary" href="{{ route('student.edit',$student->id) }}">Edit</a>
                        @csrf
                        @method('DELETE')
                        <button type="submit" class="btn btn-danger">Delete</button>
                    </form>
                </td>
            </tr>
        @endforeach
    </table>
@endsection
````

## resources\views\student\create.blade.php
````
@extends('student.layouts.app')

@section('content')
    <div class="row">
        <div class="col-lg-11">
            <h2>Add New Student</h2>
        </div>
        <div class="col-lg-1">
            <a class="btn btn-primary" href="{{ url('student') }}"> Back</a>
        </div>
    </div>

    @if ($errors->any())
        <div class="alert alert-danger">
            <strong>Whoops!</strong> There were some problems with your input.<br><br>
            <ul>
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        </div>
    @endif
    <form action="{{ route('student.store') }}" method="POST">
        @csrf
        <div class="form-group">
            <label for="txtFirstName">First Name:</label>
            <input type="text" class="form-control" id="txtFirstName" placeholder="Enter First Name" name="txtFirstName">
        </div>
        <div class="form-group">
            <label for="txtLastName">Last Name:</label>
            <input type="text" class="form-control" id="txtLastName" placeholder="Enter Last Name" name="txtLastName">
        </div>
        <div class="form-group">
            <label for="txtAddress">Address:</label>
            <textarea class="form-control" id="txtAddress" name="txtAddress" rows="10" placeholder="Enter Address"></textarea>
        </div>
        <button type="submit" class="btn btn-default">Submit</button>
    </form>
@endsection
````

## resources\views\student\view.blade.php
````
@extends('student.layouts.app')

@section('content')
    <div class="row">
        <div class="col-lg-11">
                <h2>Laravel 8 CRUD Example</h2>
        </div>
        <div class="col-lg-1">
            <a class="btn btn-primary" href="{{ url('student') }}"> Back</a>
        </div>
    </div>
    <table class="table table-bordered">
        <tr>
            <th>First Name:</th>
            <td>{{ $student->first_name }}</td>
        </tr>
        <tr>
            <th>Last Name:</th>
            <td>{{ $student->first_name }}</td>
        </tr>
        <tr>
            <th>Address:</th>
            <td>{{ $student->address }}</td>
        </tr>

    </table>
@endsection
````

## resources\views\student\edit.blade.php
````
@extends('student.layouts.app')

@section('content')
    <div class="row">
        <div class="col-lg-11">
            <h2>Update Student</h2>
        </div>
        <div class="col-lg-1">
            <a class="btn btn-primary" href="{{ url('student') }}"> Back</a>
        </div>
    </div>

    @if ($errors->any())
        <div class="alert alert-danger">
            <strong>Whoops!</strong> There were some problems with your input.<br><br>
            <ul>
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        </div>
    @endif
    <form method="post" action="{{ route('student.update',$student->id) }}" >
        @method('PATCH')
        @csrf
        <div class="form-group">
            <label for="txtFirstName">First Name:</label>
            <input type="text" class="form-control" id="txtFirstName" placeholder="Enter First Name" name="txtFirstName" value="{{ $student->first_name }}">
        </div>
        <div class="form-group">
            <label for="txtLastName">Last Name:</label>
            <input type="text" class="form-control" id="txtLastName" placeholder="Enter Last Name" name="txtLastName" value="{{ $student->last_name }}">
        </div>
        <div class="form-group">
            <label for="txtAddress">Address:</label>
            <textarea class="form-control" id="txtAddress" name="txtAddress" rows="10" placeholder="Enter Address">{{ $student->address }}</textarea>
        </div>
        <button type="submit" class="btn btn-default">Submit</button>
    </form>
@endsection
````

## run
````
php artisan serve
OR
php artisan serve --host 0.0.0.0
````

## url
````
http://127.0.0.1:8000/student
````
