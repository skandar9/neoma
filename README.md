Project overview:

The project is a web application that serves as a content management system for publication center, It manage and present different types of content, including author guidelines, publication ethics, and general programming knowledge.
The project includes contributions from multiple authors and editors
Web application, a publishing company that specializes in publishing scientific research papers, articles, and journals

I have developed an admin control panel using the Material Dashboard template and created APIs to provide data to the frontend side.

> :warning: **Warning:** This contents below ↓ contains just parts of my code.
>                        You can access my full project files by clone it from my GitLab repository
>                        (requires asking for my permissions  to grant you access to it):
>                        https://gitlab.com/skandar.s1998/neoma 

## Contents
(contains descriptive parts of my code)


[Tables and relations](#tables-and-relations)

[Project actions and progress(graph)](#project-graph)

[Login to the dashboard functionality](#dashboard-login)

[Update journal](#update-journal)

[Upload file & delete file if exist](#upload/delete-file)

[Archived manuscripts](#archived_manuscripts)

[Show specific manuscript](#show-manuscript)

[Refuse specific manuscript](#refuse-manuscript)

[Editor modal (dashboard)](#editor-modal)

[Add authors to a specific paper. *Using JQuery & JavaScript* (dashboard)](#add-authors-to-a-specific-paper)



### **tables-and-relations**

![Logo](/images/tables.png)

For more details about the content of the tables <a href="/neoma.pdf" target="_blank">Click here</a>

[🔝 Back to contents](#contents)

### **project-graph**

This graph diagram represents the actions and progress for the project.

![App Logo](/images/graph(1).png)
![App Logo](/images/graph(2).png)

[🔝 Back to contents](#contents)

### **dashboard-login**

![App Logo](/images/login.png)

I used Laravel UI package in this project that provides a convenient way to generate the frontend boilerplate code for authentication views, such as login, registration, and password reset forms.

I Run the following command to install the package:

```shell
composer require laravel/ui
```

This package I used to generate the authentication views.Specifically, I utilized the following file to facilitate the login process for the admin, granting access to their dashboard.

`resources/views/auth/login.blade.php`: This file contains the HTML template for the login form. It includes form fields for the username and password, along with any necessary validation error messages.

I depended on `Auth` Facade that is a core component of Laravel that provides convenient methods for user authentication. It is used to authenticate users and manage their sessions.

### login functionality workflow:

## Routes in `web.php`

`routes\web.php`
### Index Route

```php
Route::get('/', [PagesController::class, 'index'])->name('index');
```
This route is responsible for handling the homepage of the application. When a user visits the root URL, the `index` method of the `PagesController` will be executed.

### Dashboard Route

```php
Route::get('/dashboard', [PagesController::class, 'dashboard'])->middleware(['auth'])->name('dashboard');
```
This route represents the dashboard page of the application. It is protected by the `'auth'` middleware, which means only authenticated users can access it. When a user visits the `/dashboard` URL, the `dashboard` method of the `PagesController` will be executed.


## PagesController

`app\Http\Controllers\PagesController.php`

```php
class PagesController extends Controller
{
    public function index(){
        return redirect('/login');
    }

    public function dashboard(){
        return view('dashboard');
    }
}
```
### `index` Method

This method is responsible for handling the request to the homepage of the application. It performs a redirect to the `/login` URL. When a user visits the homepage, they will be redirected to the login page.

## Authentication Route

`routes\auth.php`

```php
Route::get('/login', [AuthenticatedSessionController::class, 'create'])
                ->middleware('guest')
                ->name('login');

Route::post('/login', [AuthenticatedSessionController::class, 'store'])
                ->middleware('guest');
```
### Login Route
This route is responsible for rendering the login form. When a GET request is made to the `/login` URL, the `create` method of the `AuthenticatedSessionController` will be executed. This route is protected by the `'guest'` middleware, which ensures that only non-authenticated users can access the login page.

### Login POST Route
This route is responsible for handling the submission of the login form. When a POST request is made to the `/login` URL, the `store` method of the `AuthenticatedSessionController` will be executed. This route is also protected by the `'guest'` middleware.

## AuthenticatedSessionController

`app\Http\Controllers\Auth\AuthenticatedSessionController.php`

```php
class AuthenticatedSessionController extends Controller
{
    public function create()
    {
        return view('auth.login');
    }

    public function store(LoginRequest $request)
    {
        $request->authenticate();

        $request->session()->regenerate();

        return redirect()->intended(RouteServiceProvider::HOME);
    }
}
```
### `create` Method
This method is responsible for displaying the login view. It returns the view named `'auth.login'`, which is the login form view. When this method is called, the login form will be rendered and displayed to the user.

### `store` Method
This method handles the incoming authentication request when the login form is submitted. It expects an instance of `LoginRequest` as a parameter, which contains the validation rules for the login request. The method calls the `authenticate` method on the request instance to perform the authentication. If the authentication is successful, the user's session is regenerated, and the user is redirected to the intended page (defined by `RouteServiceProvider::HOME`).

## Login form view

`resources\views\auth\login.blade.php`

```html
<x-layouts.guest>
    <x-auth-card>
        <x-slot name="logo">
            <a href="/">
                <img src="/img/aim.png" width="220px">
            </a>
        </x-slot>

        <!-- Session Status -->
        <x-auth-session-status class="mb-4" :status="session('status')" />

        <!-- Validation Errors -->
        <x-auth-validation-errors class="mb-4" :errors="$errors" />

        <form method="POST" action="{{ route('login') }}">
            @csrf

            <!-- Username -->
            <div>
                <x-label for="username" :value="__('Username')" />
                <x-input id="username" class="block mt-1 w-full" type="text" name="username" :value="old('username')" required autofocus />
            </div>

            <!-- Password -->
            <div class="mt-4">
                <x-label for="password" :value="__('Password')" />
                <x-input id="password" class="block mt-1 w-full"
                                type="password"
                                name="password"
                                required autocomplete="current-password" />
            </div>

            <!-- Remember Me -->
            <div class="block mt-4">
                <label for="remember_me" class="inline-flex items-center">
                    <input id="remember_me" type="checkbox" class="rounded border-gray-300 text-indigo-600 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50" name="remember">
                    <span class="ml-2 text-sm text-gray-600">{{ __('Remember me') }}</span>
                </label>
            </div>

            <div class="flex items-center justify-end mt-4">
                @if (Route::has('password.request'))
                    <a class="underline text-sm text-gray-600 hover:text-gray-900" href="{{ route('password.request') }}">
                        {{ __('Forgot your password?') }}
                    </a>
                @endif

                <x-button class="ml-3">
                    {{ __('Log in') }}
                </x-button>
            </div>
        </form>
    </x-auth-card>
</x-layouts.guest>
```

The `login.blade.php` file contains the HTML markup for the login form view.
This file is installed with the package `laravel/ui`.

[🔝 Back to contents](#contents)

## LoginRequest

`app\Http\Requests\Auth\LoginRequest.php`

The `LoginRequest` class extends the `FormRequest` class in Laravel and is responsible for handling validation and authentication for the login form submission.
### `authorize` Method
This method determines if the user is authorized to make the login request. In this case, it always returns `true`, allowing any user to make the request.

### `rules` Method
This method defines the validation rules for the login request. It specifies that the `username` and `password` fields are required and must be of type `'string'`.

### `authenticate` Method
This method attempts to authenticate the user using the provided credentials. It first ensures that the request is not rate limited by calling the `ensureIsNotRateLimited` method. Then, it uses the `Auth::attempt` method to attempt authentication using the `username` and `password` fields from the request. It also includes the `remember` value as a boolean parameter to determine if the user wants to be remembered for future sessions. If the authentication attempt fails, the method hits the rate limiter by calling `RateLimiter::hit` and throws a `ValidationException` with the error message `'auth.failed'`. If the authentication attempt succeeds, the rate limiter is cleared by calling `RateLimiter::clear`.

The `LoginRequest` class handles the validation of the login form inputs and performs the authentication logic, including rate limiting to protect against brute-force attacks.


When the login credentials are successfully matched, the user is redirected to the admin dashboard page:

![App Logo](/images/dashboard.png)


[🔝 Back to contents](#contents)

### **authentication**

routes\api.php:

```php
Route::post('register',  [AuthController::class, 'register']);
Route::post('login'   ,  [AuthController::class, 'login']);
```

app\Http\Controllers\AuthController.php:

The constructor, this function is part of a controller class and is responsible for setting up the middleware for authentication using Laravel Sanctum.
This middleware ensures that the user is authenticated using the Sanctum authentication guard before accessing the methods.
The $this->middleware('auth:sanctum')->only(['logout', 'user']); line specifies that the'auth:sanctum' middleware should be applied only to the 'logout' and 'user' methods.

```php
public function __construct()
{
    $this->middleware('auth:api')->only(['logout']);
}
```

```php
public function register (Request $request) 
{
    $request->validate([
        'name'       => ['required', 'string'],
        'email'      => ['required', 'string', 'email', 'unique:users'],  
        'password'   => ['required', 'string', 'min:6', 'confirmed'],
    ]);

    $user = User::create([
        'name'      => $request->name,
        'email'     => $request->email,
        'password'  => Hash::make($request->password),
        'role'      => 'user',
        'balance'   => 0,
    ]);

    $token = $user->createToken('Proxy App')->accessToken;
    
    return response()->json([
        'user' => new UserResource($user),
        'token' => $token,
    ], 200);
}
```

```php
public function login (Request $request) 
{
   $request->validate([
        'email'      => ['required', 'string', 'email'],  
        'password'   => ['required', 'string'],
    ]);
    $user = User::where('email', $request->email)->first();
    if ($user) {
        if (Hash::check($request->password, $user->password)) {
            $token = $user->createToken('Mega Panel App')->accessToken;
            return response()->json([
                'user' => new UserResource($user),
                'token' => $token,
            ], 200);   
        }
    }
    
    return response()->json([
        'message' => 'email or password is incorrect.',
        'errors' => [
            'email' => ['email or password is incorrect.']
        ]
    ], 422);
}
```

[🔝 Back to contents](#contents)

### **store-publication**

app\Http\Controllers\Dashboard\PublicationController.php:

the store() function validates the request data, uploads the image and files if present, creates a new publication, and redirects back with a success message.

I used various validation rules, such as ensuring that the 'title' field is an array, the 'title.ar' and 'title.en' fields are required when 'title' is present, the 'image' field is required and should be an image file, and the value of the 'type' field is either 'publications' or 'policies'.

The function then proceeds to upload the image file using the [upload_file function](#upload_file), which takes the 'image' from the request, specifies the file destination directory as 'publications_images', and returns the path of the uploaded image.

```php
public function store(Request $request)
{
    $request->validate([
        'title'          => ['array'],
        'title.ar'       => ['required_with:title'],
        'title.en'       => ['required_with:title'],
        'description'    => ['nullable','array'],
        'image'          => ['required','image'],
        'file'           => ['nullable', 'array'],
        'file.ar'        => ['nullable','mimes:pdf'],
        'file.en'        => ['nullable','mimes:pdf'],
        'file_title'     => ['nullable', 'array'],
        'type'           => ['in:publications,policies'],
    ]);
    $image = upload_file($request->image, 'image_publication', 'publications_images');
    $file_en = null;
    $file_ar = null;
    if ($request->hasFile('file.en')) {
        $file_en  = upload_file($request->file['en'], 'file-en', 'publications_en_files');
    }
    if ($request->hasFile('file.ar')) {
        $file_ar  = upload_file($request->file['ar'], 'file-ar', 'publications_ar_files');
    }
    Publication::create([
        'file_title'  => $request->file_title,
        'title'       => $request->title,
        'description' => $request->description,
        'image'       => $image,
        'file'        => [
            'en' => $file_en,
            'ar' => $file_ar,
        ],
        'type'        => $request->type,
    ]);
    return back()->withStatus(__('Publication is created successfully'));
}
```
### **upload-file**

app\helpers.php:

the upload_file() function takes a file, generates a unique filename, stores the file in the specified folder, and returns the relative path of the uploaded file,
it placed into app helper class  provide common functions for various tasks,  These helper functions are globally accessible throughout the application without needing to import or instantiate any specific class.

The function takes three parameters:
1-$request_file: The file obtained from the request.
2-$prefix: A prefix string used to create a unique filename.
3-$folder_name: The name of the folder where the file will be stored.

the function returns the relative path of the uploaded file by concatenating the $folder_name and the unique filename. This path can be used to reference and retrieve the file later.

```php
function upload_file($request_file, $prefix, $folder_name)
{
    $extension = $request_file->getClientOriginalExtension();
    $file_to_store = $prefix . '_' . time() . rand(10000, 99999) . '.' . $extension;
    $request_file->storeAs('public/' . $folder_name, $file_to_store);
    return $folder_name . '/' . $file_to_store;
}
```

[🔝 Back to contents](#contents)

### **update-journal**

`app\Http\Controllers\JournalController.php`

```php
public function update(Request $request, Journal $journal)
{
    $request->validate([
        'slug'           => ['string',Rule::unique('journals', 'slug')->ignore($journal->id)],
        'category_id'    => ['exists:categories,id'],
        'frequency'      => ['in:monthly,quarterly,yearly,semi_annually,irregular'],
        'fee'            => ['nullable','numeric'],
        'cover_image'    => ['image','nullable'],
        'issn'           => ['nullable','regex:([0-9]{4}\-[0-9]{4})',Rule::unique('journals', 'issn')->ignore($journal->id)],
        'keywords'       => ['array','nullable'],
        'indexing_ids'   => ['array', 'nullable'],
        'indexing_ids*'  => ['exists:indexing_entities,id'],
    ]);

    $cover_image = $journal->cover_image;

    $keywords = json_encode($request->keywords);

    if($request->hasFile('cover_image'))
    {
        delete_file_if_exist($cover_image);
        $cover_image = upload_file($request->cover_image,'journal','journals_images');
    }

    $journal->update([
        'title'        => $request->title?? $journal->title,
        'slug'         => $request->slug?? $journal->slug,
        'description'  => $request->description?? $journal->description,
        'category_id'  => $request->category_id?? $journal->category_id,
        'frequency'    => $request->frequency?? $journal->frequency,
        'fee'          => $request->fee?? $journal->fee,
        'issn'         => $request->issn?? $journal->issn,
        'aim'          => $request->aim?? $journal->aim,
        'ethics'       => $request->ethics?? $journal->ethics,
        'guideline'    => $request->guideline?? $journal->guideline,
        'cover_image'  => $cover_image,
        'key_words'    => $keywords,
    ]);

    if($request->indexing_ids)
    {
        $indexing_entities = JournalIndexing::where('journal_id', $journal->id)->get();
        foreach($indexing_entities as $entity)
            $entity->delete();

        foreach($request->indexing_ids as $indexing){
            JournalIndexing::create([
                'journal_id'          => $journal->id,
                'indexing_entity_id'  => $indexing,
            ]);
        }
    }
    return back()->withStatus(__('Journal is updated successfully'));
}
```
The `update` function represents a form for updating journal information. It handles the validation of the incoming request, updates the journal entry in the database, and manages the deletion and creation of JournalIndexing entities associated with the journal.

This function is called from the following route defined in the `routes\web.php` file:
```php
Route::Resource('/dashboard/journals', JournalController::class)->except(['show']);
```
Explain for some fields included inside block that validates the request data using the `validate` method:

- `slug`: Should be a string and must be unique in the journals table, except when updating
   the current journal(`ignore($journal->id)`   is used to ignore the current journal's ID during uniqueness check).
- `category_id`: Should exist in the categories table.
- `frequency`: Should have one of the values: monthly, quarterly, yearly, semi_annually, or irregular.
- `issn` (represents the International Standard Serial Number): Optional, but if provided,
   it should match the regex pattern `([0-9]{4}\-[0-9]{4})`. This pattern requires the `issn` to consist of eight digits separated by a hyphen (e.g., 1234-5678) and must be unique in the journals table, except when updating the current journal.

The code may call two methods, [Upload file & delete file if exist](#upload/delete-file), from the helper class.

If the request contains `indexing_ids`, it performs the following steps:

1. It retrieves all JournalIndexing entities associated with the current journal by querying the JournalIndexing model where the `journal_id` matches the ID of the current journal.
2. It iterates over the retrieved `indexing_entities` and deletes them one by one using the `delete` method.
3. Then, it iterates over the `indexing_ids` array from the request and creates new JournalIndexing entries for each value. Each entry is associated with the current journal (using `journal_id`) and the corresponding indexing entity ID (using `indexing_entity_id`).

[🔝 Back to contents](#contents)

### **upload/delete-file**

`app\helpers.php`

```php
.
.
function upload_file($request_file, $prefix, $folder_name)
{
    $extension = $request_file->getClientOriginalExtension();
    $file_to_store = $prefix . '_' . time() . '.' . $extension;
    $request_file->storeAs('public/' . $folder_name, $file_to_store);
    return $folder_name.'/'.$file_to_store;
}
.
.
function delete_file_if_exist($file)
{
    if(Storage::exists('public/'.$file))
        Storage::delete('public/'.$file);
}
```

The `upload_file` function is a utility function that handles file uploads within the application. It performs the following tasks:

1. Takes a file as input and generates a unique filename for storage.
2. Stores the file in the specified folder.
3. Returns the relative path of the uploaded file.

It takes three parameters:

1. `$request_file`: The file obtained from the request.
2. `$prefix`: A prefix string used to create a unique filename.
3. `$folder_name`: The name of the folder where the file will be stored.

The function returns the relative path of the uploaded file. This path is obtained by concatenating the `$folder_name` and the unique filename generated for the file.

The `upload_file` function is placed within the app helper class, which provides common functions for various tasks. These helper functions are globally accessible throughout the application without needing to import or instantiate any specific class.

[🔝 Back to contents](#contents)

### **archived_manuscripts**

`app\Http\Controllers\Dashboard\ManuscriptController.php`

```php
public function archived_manuscripts()
{
    // Fetch manuscripts associated with the logged-in user (editor) that have a status other than "pending"
    $manuscripts = to_user(Auth::user())->editor_manuscripts()->where('manuscript_editors.status', '!=', 'pending')->get();
    $type = 'archived';
    return view('dashboard.manuscript_view')->with(['manuscripts' => $manuscripts, 'type' => $type]);
}
```
![archived manuscripts page from dashboard](/images/archived_manuscript.png)

The purpose of the `archived_manuscripts()` method is to fetch manuscripts associated with the logged-in user (editor) and render the 'manuscript_view' page (as shown in the picture) with the retrieved manuscripts. Only manuscripts that have a status other than "pending" (i.e., archived manuscripts/refused manuscripts) are included. The view can then display this data to the user and provide them with the ability to perform further actions on the manuscripts.
This method is called from the following route defined in the `routes\web.php` file:
```php
Route::get('/dashboard/manuscripts/archived', [ManuscriptController::class, 'archived_manuscripts'])->name('archived');
```
The [editor_manuscripts()](#editor_manuscripts()-relationship) is a relationship defined on the User model that represents the manuscripts assigned to the editor.

By clicking on the 'info' icon (as shown in the picture), it redirects to the detail page of a specific manuscript by calling the ["Show manuscript" function](#show-manuscript).

[🔝 Back to contents](#contents)

### **editor_manuscripts()-relationship**

`app\Models\User.php`

```php
.
.
.
public function editor_manuscripts()
{
    // Establishes a many-to-many relationship between the User and Manuscript models
    return $this->belongsToMany(Manuscript::class, 'manuscript_editors', 'editor_id', 'manuscript_id');
}
```

The `editor_manuscripts()` method defines a many-to-many relationship between the User model and the Manuscript model. This relationship allows retrieving the manuscripts associated with a specific editor.

- The `'manuscript_editors'` argument specifies the name of the intermediate table that connects the two models. This table is used to store the relationship between editors and manuscripts.

- The `'editor_id'` argument specifies the foreign key column in the intermediate table that corresponds to the User model.

- The `'manuscript_id'` argument specifies the foreign key column in the intermediate table that corresponds to the Manuscript model.

[🔝 Back to contents](#contents)

### **show Manuscript**

`app\Http\Controllers\Dashboard\ManuscriptController.php`

```php
public function show($uuid)
{
    $manuscript = Manuscript::where('uuid', $uuid)->firstOrFail();

    // Check if the authenticated user is an editor associated with the same journal as the manuscript
    $editor = Editor::where([
        ['user_id', '=', Auth::user()->id],
        ['journal_id', '=', $manuscript->journal_id],
    ])->first();

    if (!$editor) {
        abort(403);
    }

    // Retrieve additional data related to the manuscript
    $type = $editor->type;
    $editors = Editor::where('type', 'editor')->where('journal_id', $manuscript->journal_id)->get();
    $volumes = Volume::where('journal_id', $manuscript->journal_id)->get();
    $volumes_ids = $volumes->pluck('id');
    $issues = Issue::whereIn('volume_id', $volumes_ids)->get();

    return view('dashboard.manuscript_details')->with([
        'manuscript' => $manuscript,
        'type' => $type,
        'editors' => $editors,
        'issues' => $issues,
        'volumes' => $volumes,
    ]);
}
```

The `show()` method fetches the details of a specific manuscript identified by its UUID. It performs the following steps:

1. Retrieves the manuscript from the database using the UUID provided.
2. Checks if the authenticated user is an editor associated with the same journal as the manuscript by querying the Editor model.
3. If the user is not an editor for the journal, it aborts with a 403 (Forbidden) error.
4. Retrieves additional data related to the manuscript, including the editor's type, all editors associated with the journal, volumes associated with the journal, and issues associated with the volumes.
5. Renders the 'manuscript_details' view and passes the retrieved data to be displayed in the view.

The 'manuscript_details' view can then utilize the provided data to present the manuscript details to the user.

[🔝 Back to contents](#contents)

### **Refuse Manuscript**

`app\Http\Controllers\Dashboard\ManuscriptController.php`

```php
use App\Jobs\SendEmailsJob;
use App\Mail\RefuseManuscriptMail;

public function refuse(Request $request, $uuid)
{
    // Retrieve the manuscript by UUID
    $manuscript = Manuscript::where('uuid', $uuid)->firstOrFail();

    // Retrieve the author associated with the manuscript
    $author = Author::where([
        ['article_type', '=', Manuscript::class],
        ['article_id', '=', $manuscript->id],
    ])->first();

    $manuscript->update([
        'status' => 'refused',
    ]);

    // Retrieve the ManuscriptEditor record for the editor and manuscript
    $manuscript_editor = ManuscriptEditor::where([
        ['manuscript_id', '=', $manuscript->id],
        ['editor_id', '=', Auth::user()->id],
    ])->first();

    $manuscript_editor->update([
        'letter' => $request->reason,
        'status' => 'rejected'
    ]);

    $details = [
        'name' => $author->first_name,
        'manuscript_uuid' => $manuscript->uuid,
        'letter' => $request->reason
    ];

    $emailJob = new SendEmailsJob($author->email, RefuseManuscriptMail::class, $details);
    $this->dispatch($emailJob);

    return back();
}
```

The `refuse()` method handles the refusal of a manuscript by the editor. It performs the following steps:

1. Retrieve the manuscript from the database using the UUID provided.
2. Retrieve the author associated with the manuscript.
3. Update the manuscript's status to 'refused'.
4. Retrieve the ManuscriptEditor record for the editor and manuscript.
5. Update the ManuscriptEditor record with the refusal reasons provided in the request.
6. Prepare the details for the email notification, including the author's name, manuscript UUID, and refusal reasons.
7. Create an instance of the `SendEmailsJob` and dispatch it to send the email to the author.
8. Redirect back to the previous page.

[🔝 Back to contents](#contents)

### **editor-modal**

![App Logo](/images/editor-modal.png)

`resources\views\dashboard\editor_view.blade.php`

```html
.
.
.
.
    <button type="button" class="btn btn-success" style="margin-left: auto;display:block;" data-toggle="modal" data-target="#addModal"> {{ __('Add') }} </button>
    <!-- Add Modal -->
    <div class="modal fade" id="addModal" role="dialog" aria-labelledby="addModalTitle" aria-hidden="true">
    <div class="modal-dialog modal-dialog-centered" role="document">
        <div class="modal-content">
        <div class="modal-header">
            <h5 class="modal-title" id="addModalLabel">Add editor</h5>
            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                <span aria-hidden="true">&times;</span>
                </button>
            </div>
            <div class="modal-body">
            <div id="add-error" class="alert alert-danger" style="color: red;display:none;">The entered values ​​are not valid</div>
            <form method="post" action="{{ route('journal_editors.editors.store', [$journal->id]) }}" autocomplete="off" class="form-horizontal" enctype="multipart/form-data">
                @csrf
                <div class="row">
                    <label class="col-sm-3 col-form-label" for="input-user">{{ __('Editor') }}</label>
                    <div class="col-sm-8">
                        <div class="form-group{{ $errors->has('user') ? ' has-danger' : '' }}">
                            <select class="form-control{{ $errors->has('user_id') ? ' is-invalid' : '' }}" name="user_id" id="input-user-id" required style="width:100%">
                                <option value="">Select Editor</option>
                                @foreach ($users as $user)
                                    <option value="{{ $user->id }}" @if ($user->id  ==  old('user_id') ) selected @endif> {{ $user->first_name }} {{ $user->last_name }} ({{ $user->email }})</option>
                                @endforeach
                            </select>
                            @if ($errors->has('type'))
                                <span id="type-error" class="error text-danger" for="input-type">{{ $errors->first('type') }}</span>
                            @endif
                        </div>
                    </div>
                </div>
                <div class="row">
                    <label class="col-sm-3 col-form-label" for="input-type">{{ __('Type') }}</label>
                    <div class="col-sm-8">
                        <div class="form-group{{ $errors->has('type') ? ' has-danger' : '' }}">
                            <select class="form-control{{ $errors->has('type') ? ' is-invalid' : '' }}" name="type" id="input-type" required>
                                <option value="chief" @if (old('type')=="chief") selected @endif>Editor In Chief</option>
                                <option value="editor" @if (old('type')=="chief") selected @endif>Editor</option>
                                <option value="managing" @if (old('type')=="chief") selected @endif>Managing Editor</option>
                            </select>
                            @if ($errors->has('type'))
                                <span id="type-error" class="error text-danger" for="input-type">{{ $errors->first('type') }}</span>
                            @endif
                        </div>
                    </div>
                </div>
                <div class="modal-footer">
                    <button type="submit" class="btn btn-success">Add</button>
                </div>
            </form>
            </div>
        </div>
    </div>
    </div>
    <!-- End Modal -->
.
.
.
.
```

This HTML code represents a modal dialog for adding an editor.

The modal used in this section has the following attributes and components:

- Unique ID: The modal has a unique `id` attribute set to "addModal" to differentiate it from other modals on the page.
- Role: The `role` attribute is set to "dialog" to define the modal as a dialog box.
- Aria-labelledby: The `aria-labelledby` attribute is used to associate the modal with the element that labels it. In this case, the label is specified by the "addModalTitle" ID.

### Modal Header

The modal header includes the following elements:

- Title: The `modal-header` class is added to the modal content, displaying the title as "Add editor."
- Close button: A close button is represented by the `button` element with the class `close`. It has a data attribute `data-dismiss="modal"` to enable the closing functionality. The `aria-label` attribute is provided for accessibility purposes.

### Modal Body

The modal body, included in the modal content, contains the form for adding an editor. The form is defined within a `<form>` element with the following attributes:

- Method: The `method` attribute is set to "post" to specify that the form data will be submitted using the HTTP POST method.
- Action: The `action` attribute specifies the route where the form data will be submitted.
- CSRF Protection: The form utilizes CSRF protection with the `@csrf` directive to ensure secure data submission.

The form consists of two sections:

1. First Section:
   - Label: The first section includes a label for the "Editor" field.
   - Select Element: A `<select>` element with the name "user_id" is present, allowing the user to select an editor from a list of  options. The options are dynamically generated using a `foreach` loop and populated with data from the `$users` variable. The selected option is determined based on the value of the "user_id" stored in the old input.
   - Select2 Library: The Select2 library is used to enhance the functionality of the select element, providing features like searching, tagging, and customization. To utilize Select2, the necessary CSS and JavaScript files are included at the top of the page:
     ```html
     <link href="https://cdn.jsdelivr.net/npm/select2@4.1.0-rc.0/dist/css/select2.min.css" rel="stylesheet" />
     <script src="https://cdn.jsdelivr.net/npm/select2@4.1.0-rc.0/dist/js/select2.min.js"></script>
     ```

2. Second Section:
   - Label: The second section contains a label for the "Type" field.
   - Select Element: Another `<select>` element with the name "type" allows the user to select the type of editor from the available options: "Editor In Chief," "Editor," or "Managing Editor." Similar to the previous section, the selected option is determined based on the value of the "type" stored in the old input (if any).

[🔝 Back to contents](#contents)

### **add-authors-to-a-specific-paper**

![App Logo](/images/add-authors.png)

This section provides a form for adding authors to a specific paper. The form allows users to input information such as the author's first name, last name, email, affiliation, and country.

`resources\views\dashboard\paper_create.blade.php`

```html
<x-layouts.dashboard>
.
.
.
    <form method="post" action="{{ route('papers.store') }}" autocomplete="off"        
        enctype="multipart/form-data">
        @csrf
        <div class="row">
            <div class="col-12">
                <div class="card">
                    <div class="card-body">
                        <div class="form-body">
                            <div class="row">
                                <label>Add Authors</label>
                                <div class="form-group" style="width: 100%;overflow-x:auto">
                                    <div class="float-child">
                                        <div class="green">
                                            <button type="button" class="btn btn-outline-primary" name="button" id="add-button" style="font-size: 15px; width: 60px; padding: 2px 5px; ">Add</button>
                                            <table>
                                                <tbody id="authors-list">
                                                    <tr class="author">
                                                        <td><input type="text" placeholder="First Name" name="first_names[]" class="arabic" size="25"></td>
                                                        <td><input type="text" placeholder="Last Name" name="last_names[]" class="arabic" size="25"></td>
                                                        <td><input type="email" placeholder="Email" name="emails[]" class="arabic" size="25"></td>
                                                        <td><input type="text" placeholder="Affiliation" name="affiliations[]" class="arabic" size="20"></td>
                                                        <td>
                                                            <select class="arabic" style="width: 135px; height: 30px;" name="countries[]">
                                                                <option value={{ null }}>Select Country</option>
                                                                @foreach ($countries as $country)
                                                                    <option value="{{ $country->id }}">{{ $country->translations['name']['en'] }}</option>
                                                                @endforeach
                                                            </select>
                                                        </td>
                                                        <td><button type="button" class="btn btn-outline-danger" name="button"  style="font-size: 10px; text-transform:lowercase ; padding: 8px 10px 8px 10px; " onclick="deleteEntry(this)"><i class="icon fa-regular fa-trash-can "></i></button></td>
                                                    </tr>
                                                </tbody>
                                            </table>
                                            @if ($errors->has('author'))
                                                <span id="author_error" class="error text-danger"
                                                    for="input_author">{{ $errors->first('author') }}</span>
                                                <br>
                                            @endif
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <button type="submit" class="btn btn-success">{{ __('Save') }}</button>
    </form>

    <script src='https://cdnjs.cloudflare.com/ajax/libs/jquery/3.2.1/jquery.min.js'></script>
    <script>
        jQuery(function($) {
            var authors_list = $('#authors-list');
            var author_row = authors_list.find('.author:first').clone();

            $('#add-button').click(function() {
                authors_list.append(author_row.clone());
                authors_list.find('.author:last select')[0].value = '';
                var inputs=authors_list.find('.author:last input');
                for (let i = 0; i < inputs.length; i++)
                    inputs[i].value = '';
            });
        });
    </script>
    <script>
        function deleteEntry(entry){
            var grandParent = entry.parentElement;
            grandParent.parentElement.remove();
        }
    </script>
</x-layouts.dashboard>
```

The form includes a section for adding authors to the paper. It consists of a table with input fields for the author's **first name**, **last name**, **email**, **affiliation**, and a dropdown menu for selecting the **country**.

There is an "Add" button with the id "add-button". When clicked, it clones the first author row and appends it to the table of authors' list.
Addition of new author depending on `JavaScript` code includes a `jQuery` function that handles the addition of new author rows. It clones the first author row and appends it to the table. It also clears the input fields in the cloned row.

Each author row has a delete button with an onclick attribute that triggers the `deleteEntry() `function. This function removes the corresponding author row from the table.

## Adding Authors

To add authors, click the "Add" button with the id "add-button". This button clones the first author row and appends it to the table of authors' list. The cloned row will have all input fields cleared and ready for new author information.

## Removing Authors

Each author row has a delete button with an onclick attribute that triggers the `deleteEntry()` function. This function removes the corresponding author row from the table.

## Initialization

The first script includes the jQuery library by loading it from the CDN (Content Delivery Network). This allows the use of jQuery functions and selectors in the script.

The `jQuery(function($){})` syntax is a shortcut for the `$(document).ready()` function, which ensures that the script is executed when the DOM (Document Object Model) is fully loaded and ready to be manipulated.

I included the jQuery library by adding the following script tag before the closing `</body>` tag:

```html
<script src='https://cdnjs.cloudflare.com/ajax/libs/jquery/3.2.1/jquery.min.js'></script>
```

The script initializes variables `authors_list` and `author_row`.

- `authors_list` is assigned the jQuery object representing the element with the ID `authors-list`, which is the table body where author rows are added.
- `author_row` is assigned the cloned version of the first author row in the table.

## Adding New Authors

The `$('#add-button').click()` function is bound to the click event of the element with the ID `add-button` (the "Add" button). When this button is clicked, the function is executed.

- It appends a cloned version of `author_row` to the `authors_list` table.
- It sets the value of the last select element within the appended row to an empty string.
- It retrieves all input elements within the appended row and sets their values to empty strings using a loop.

## Deleting Authors

The second script defines the `deleteEntry(entry)` function, which is called when the delete button of an author row is clicked. The function is responsible for removing the corresponding author row from the table.

- The `entry` parameter represents the delete button element that triggered the function.
- The function finds the grandparent element of the delete button (the table row) and removes it from the DOM.


[🔝 Back to contents](#contents)
