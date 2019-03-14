// config/filesystems.php
'uploads' => [
    'driver' => 'local',
    'root' => storage_path().'/files/uploads',
],
// no arquivo route/web
Route::resource('fileupload', 'FileUploadController');
// criar controller
php artisan make:controller FileUploadController --resource
// function index
public function index()
{
    $files = FileUpload::all();
    if($files){
        return response()->json(['success' => $files], 200);
    }
    return response()->json([
        'success' => false
    ], 500);

}
// function store
public function store(Request $request)
{
$file = $request->get('file');
$name = $file->getClientOriginalName();

$path = hash( 'sha256', time());

if(Storage::disk('uploads')->put($path.'/'.$name,  File::get($file))) {
    $input['name'] = $name;
    $input['mime'] = $file->getClientMimeType();
    $input['path'] = $path;
    $input['size'] = $file->getClientSize();
    $file = FileEntry::create($input);

    return response()->json([
        'success' => true,
        'id' => $file->id
    ], 200);
}
return response()->json([
    'success' => false
], 500);
}
php artisan make:model FileUpload -m
// na model
class FileUpload extends Model
{
    protected $fillable = ['name', 'mime', 'path', 'size'];
}
//na migration
public function up()
{
    Schema::create('file_uploads', function (Blueprint $table) {
        $table->increments('id');
        $table->string('name');
        $table->string('mime');
        $table->string('path');
        $table->integer('size');
        $table->timestamps();
    });
}

/**
    * Reverse the migrations.
    *
    * @return void
    */
public function down()
{
    Schema::dropIfExists('file_uploads');
}
// criar a tabela
php artisan migrate

Baseado em:
https://medium.com/@arthursorriso/upload-de-arquivos-com-laravel-e-vuejs-9317cc0097c4
https://appdividend.com/2018/02/13/vue-js-laravel-file-upload-tutorial/
