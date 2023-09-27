The project is a web application that serves as a content management system for .. . It manage and present different types of content, including author guidelines, publication ethics, and general programming knowledge.
The project includes contributions from multiple authors and editors
Web application, a publishing company that specializes in publishing scientific research papers, articles, and journals

The GeneralContentController class acts as the central controller for handling the application's general content management. It requires user authentication for accessing and modifying content, ensuring a secure environment for content management.

The controller includes methods for retrieving and updating author guidelines and publication ethics content. The "get_author_guidelines" method retrieves the author guidelines from the database and renders a view to display them. The "upsert_author_guidelines" method allows authorized users to update the author guidelines by accepting the updated content and validating it. If the author guidelines exist in the database, the method updates them; otherwise, it creates a new entry.

Similarly, the "get_publication_ethics" method retrieves the publication ethics content from the database and renders a view to display it. The "upsert_publication_ethics" method allows authorized users to update the publication ethics content by accepting the updated content and validating it. It follows a similar process of updating or creating a new entry depending on the existence of the content in the database.

The GeneralContentController class interacts with the GeneralContent model, which represents the database table storing the general content. It uses the "type" field to distinguish between different types of content, such as author guidelines and publication ethics. The model provides methods for retrieving and updating the content entries.

The project emphasizes good documentation practices and highlights the importance of the README.md file. The README.md file serves as the ultimate guide for open-source projects, providing comprehensive documentation. It covers essential aspects of the project, including setup instructions, usage guidelines, and other relevant information.

## Contents [contains parts of my code]

[Authentication](#authentication)

["Update journal" function](#update-journal)

[Upload file & delete file if exist](#upload/delete-file)

[Archived manuscripts](#archived_manuscripts)

["Show manuscript" function](#show-manuscript)

[Refuse manuscript](#refuse-manuscript)

['Editor' modal (dashboard)](#editor-modal)

[Add authors to specific paper. *Using JQuery & JavaScript* (dashboard)](#authors)

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

app\Http\Controllers\JournalController.php:

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
This *update* function represents a form for updating journal information. It handles the validation of the incoming request, updates the journal entry in the database, and manages the deletion and creation of JournalIndexing entities associated with the journal.
It called from this route that I defined into *routes\web.php* file:
```php 
Route::Resource('/dashboard/journals', JournalController::class)->except(['show']);
```
Explain for some fields included inside block that validates the request data using the validate method:
- slug: should be a string and must be unique in the journals table, except when updating the current journal (ignore($journal->id) is used to ignore the current journal's ID during uniqueness check).
- 'category_id' should exist in the categories table.
- 'frequency' should have one of the values: monthly, quarterly, yearly, semi_annually, or irregular.
- 'issn' (represents the International Standard Serial Number) is optional but, if provided,
  should match the regex pattern ([0-9]{4}\-[0-9]{4}), this pattern requires the 'issn' to consist of eight digits separated by a hyphen (e.g., 1234-5678), and must be unique in the journals table, except when updating the current journal.

The code maybe call two methods, [Upload file & delete file if exist](#upload/delete-file) from helper class.

If the request contains *indexing_ids*, it performs the following steps:
- It retrieves all JournalIndexing entities associated with the current journal
  by querying the JournalIndexing model where the journal_id matches the ID of the current journal.
- It iterates over the retrieved indexing_entities and deletes them one by one using the delete method.
- then, it iterates over the *indexing_ids* array from the request and creates new JournalIndexing entries for each value.
  Each entry is associated with the current journal (using journal_id) and the corresponding indexing entity ID (using indexing_entity_id).

[🔝 Back to contents](#contents)

### **upload/delete-file**

app\helpers.php:

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

[🔝 Back to contents](#contents)

### **archived_manuscripts**

app\Http\Controllers\Dashboard\ManuscriptController.php:

```php
public function archived_manuscripts()
{
    $manuscripts = to_user(Auth::user())->editor_manuscripts()->where('manuscript_editors.status', '!=', 'pending')->get();
    $type = 'archived';
    return view('dashboard.manuscript_view')->with(['manuscripts' => $manuscripts, 'type' => $type]);
}
```
![archived manuscripts page from dashboard](/images/archived_manuscript.png)

The purpose of *archived_manuscripts()* method is to fetch manuscripts associated with the logged-in user (editor) and render a *'manuscript_view'* (shown in the picture) with the manuscripts and that have a status other than "pending" (i.e., archived manuscripts/refused manuscripts). The view can then display this data to the user, potentially allowing them to take further actions on the manuscripts.
It called from this route that I defined into *routes\web.php* file:
```php 
Route::get('/dashboard/manuscripts/archived', [ManuscriptController::class, 'archived_manuscripts'])->name('archived');
```
The [editor_manuscripts()](#user) is a relationship defined on the User model that represents the manuscripts assigned to the editor.

By clicking on the 'info' icon (as shown in the picture), it redirects to the detail page of a specific manuscript by calling the ["Show manuscript" function](#show-manuscript).

[🔝 Back to contents](#contents)

### **user**

app\Models\User.php:

```php
.
.
.
public function editor_manuscripts(){
    return $this->belongsToMany(Manuscript::class,'manuscript_editors','editor_id','manuscript_id');
}
```
The *editor_manuscripts()* method establishes a many-to-many relationship between the user and the Manuscript. This relationship allows to retrieve the manuscripts associated with an specific editor.

*'manuscript_editors'* argument, specifies the name of the intermediate table that connects the two models. This table I used to store the relationship between editors and manuscripts.

*'editor_id'* argument specifies the foreign key column in the intermediate table that corresponds to the User model.

*'manuscript_id'* argument specifies the foreign key column in the intermediate table that corresponds to the related(Manuscript) model.

[🔝 Back to contents](#contents)

### **show-manuscript**

app\Http\Controllers\Dashboard\ManuscriptController.php:

```php
public function show($uuid)
{
    $manuscript = Manuscript::where('uuid', $uuid)->firstOrFail();
    $editor = Editor::where([
        ['user_id', '=', Auth::user()->id],
        ['journal_id', '=', $manuscript->journal_id],
    ])->first();

    if (!$editor)
        abort(403);

    $type = $editor->type;
    $editors = Editor::where('type', 'editor')->where('journal_id', $manuscript->journal_id)->get();
    $volumes = Volume::where('journal_id', $manuscript->journal_id)->get();
    $volumes_ids = $volumes->pluck('id');
    $issues = Issue::whereIn('volume_id', $volumes_ids)->get();

    return view('dashboard.manuscript_details')->with([
        'manuscript' => $manuscript,
        'type'       => $type,
        'editors'    => $editors,
        'issues'     => $issues,
        'volumes'    => $volumes,
        'editors'    => $editors,
    ]);
}
```
manuscript_details page:
![manuscript details page from dashboard](/images/manuscript_details.png)

The purpose of *show* method is to display the details of a specific manuscript identified by its UUID. It ensures that the authenticated user is an editor associated with the same journal as the manuscript. Then, it retrieves data related to the manuscript(the associated editors, volumes, and issues). Finally, it renders a *'manuscript_details'* view and passes the retrieved data to be displayed in the view (shown in the picture).

It retrieves an editor associated with the current user and the same journal as the manuscript. It searches for an Editor model where the user_id is equal to the authenticated user's ID *(Auth::user()->id)* and the journal_id matches the journal_id of the retrieved manuscript.

[🔝 Back to contents](#contents)

### **refuse-manuscript**

app\Http\Controllers\Dashboard\ManuscriptController.php:

```php
public function refuse(Request $request, $uuid)
{
    $manuscript = Manuscript::where('uuid', $uuid)->firstOrFail();
    $author = Author::where([
        ['article_type', '=', Manuscript::class],
        ['article_id', '=', $manuscript->id],
    ])->first();

    $manuscript->update([
        'status' => 'refused',
    ]);

    $manuscript_editor = ManuscriptEditor::where([
        ['manuscript_id', '=', $manuscript->id],
        ['editor_id', '=', Auth::user()->id],
    ])->first();

    $manuscript_editor->update([
        'letter' => $request->reason,
        'status' => 'rejected'
    ]);

    $details = [
        'name'            => $author->first_name,
        'manuscript_uuid' => $manuscript->uuid,
        'letter'          => $request->reason
    ];

    $emailJob = new SendEmailsJop($author->email, "App\Mail\RefuseManuscriptMail", $details);
    $this->dispatch($emailJob);
    return back();
}
```
The purpose of *refuse* method is to handle the refusal of a manuscript by editor of it. It updates the manuscript's status and the corresponding ManuscriptEditor record with the refusal reasons provided in the request.
It then sends an email to the author notifying them about the refusal, by creates an instance of the *SendEmailsJop* job, passing the author's email, the class name of the RefuseManuscriptMail Mailable, and the *$details* array as parameters.

[🔝 Back to contents](#contents)

### **editor-modal**

![Editor modal](/images/editor-modal.png)

resources\views\dashboard\editor_view.blade.php:

This HTML code represents a modal dialog for adding an editor.

The modal has a unique id attribute set to "addModal" and defines its role as a dialog. The aria-labelledby attribute is used to associate the modal with the element that labels it, specified by the "addModalTitle" ID.

The class modal-header (which included into modal content), It includes a title displayed as "Add editor" and a close button (represented by the button element) with the class close. The close button has a data attribute data-dismiss="modal" and an aria-label attribute for accessibility.

The class modal-body (which included into modal content). It contains the form for adding an editor.
The form for adding an editor is defined within a <form> element. It has the method attribute set to "post" and an action attribute that specifies the route where the form data will be submitted. The form uses  CSRF protection with @csrf.

The form includes two sections:
- The first section contains a label for the "Editor" field and a <select>
  element with the name "user_id". It allows the user to select an editor from a  list of options. The options are generated dynamically using a foreach loop and  populated with data from the $users variable. The selected option is determined  based on the value of the "user_id" stored in the old input.
  here I used thr Select2 library, which is a popular JavaScript library for enhancing select elements (dropdowns) with features like searching, tagging, and customization, and for that I included both the CSS and JavaScript files at the top of the page:
  <link href="https://cdn.jsdelivr.net/npm/select2@4.1.0-rc.0/dist/css/select2.min.css" rel="stylesheet" />
  <script src="https://cdn.jsdelivr.net/npm/select2@4.1.0-rc.0/dist/js/select2.min.js"></script>

- The second section contains a label for the "Type" field and a <select>
  element with the name "type".It allows the user to select the type of editor from the available options"Editor In Chief," "Editor," or "Managing Editor." Similar to the previous section, the selected option is determined based on the value of the "type" stored in the old input (if any).

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
[🔝 Back to contents](#contents)

### **authors**

resources\views\dashboard\paper_create.blade.php:

The provided code represents form for adding authors to a specific paper.

The form includes a section for adding authors to the paper. It consists of a table with input fields for the author's *first name*, *last name*, *email*, *affiliation*, and a dropdown menu for selecting the *country*.

There is an "Add" button with the id "add-button". When clicked, it clones the first author row and appends it to the table of authors' list.
Addition of new author depending on JavaScript code includes a jQuery function that handles the addition of new author rows. It clones the first author row and appends it to the table. It also clears the input fields in the cloned row.

Each author row has a delete button with an onclick attribute that triggers the deleteEntry() function. This function removes the corresponding author row from the table.

The first script includes the jQuery library by loading it from the CDN (Content Delivery Network). This allows the use of jQuery functions and selectors in the script.

The jQuery(function($){}) syntax is a shortcut for the $(document).ready() function, which ensures that the script is executed when the DOM (Document Object Model) is fully loaded and ready to be manipulated.

The script initializes variables authors_list and author_row.

authors_list is assigned the jQuery object representing the element with the ID authors-list, which is the table body where author rows are added.
author_row is assigned the cloned version of the first author row in the table.
The $('#add-button').click() function is bound to the click event of the element with the ID add-button (the "Add" button). When this button is clicked, the function is executed.

It appends a cloned version of author_row to the authors_list table.
It sets the value of the last select element within the appended row to an empty string.
It retrieves all input elements within the appended row and sets their values to empty strings using a loop.
The second script defines the deleteEntry(entry) function, which is called when the delete button of an author row is clicked. The function is responsible for removing the corresponding author row from the table.

The entry parameter represents the delete button element that triggered the function.
The function finds the grandparent element of the delete button (the table row) and removes it from the DOM.
Overall, these scripts

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

[🔝 Back to contents](#contents)
