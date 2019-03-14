# FIleUpload Laravel Vue
## Upload de arquivos com componente Vue e Axios 

### Exemplo de utilização no Laravel >= 5.5.* e Vue 2

Pré requisitos:
Laravel >= 5.5.*
Bootstrap 4 (apenas o css)  
Font-Awesome (para os icones)

## Configuração do Laravel:

Para começar podemos configurar um disco em **config/filesystems.php**

```
    'uploads' => [
        'driver' => 'local',
        'root' => storage_path().'/files/uploads',
    ],
```

Se estiver usando o servidor artisan, reinicie para pegar essa nova configuração  
Caso seja outro servidor rode o comando:

```
php artisan config:cache
```

Para poder acessar os dados eles devem ser visíveis apenas na pasta public, para não comprometer a segurança crie um link simbólico para de /storage para /public com o comando:

```
php artisan storage:link
```

As rotas em **route/web**

```
Route::resource('fileupload', 'FileUploadController');
```
Criar a Model:

```
php artisan make:model FileUpload -m
````

Na model deve ter os filables para poder salvar os dados:

```
class FileUpload extends Model
{
    protected $fillable = ['name', 'mime', 'path', 'size'];
}
```

O comando vai criar uma Model e uma migration, na migration você pode fazer como abaixo

```
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

public function down()
{
    Schema::dropIfExists('file_uploads');
}
```

Rode o comando para criar a tabela:

```
php artisan migrate
```

Criar controller

```
php artisan make:controller FileUploadController --resource
```

Um exemplo de controller

```
use File;
use Response;
Use App\FileUpload;

class FileUploadController extends Controller
{
    public function index($id)
    {
        $list = Task::find($id)->images;
        return response()->json($list);
    }

    public function show($id)
    {
        $search = FileUpload::find($id);
        $path = storage_path('files/uploads/'.$search->path.'/'.$search->name);
        if (!File::exists($path)) {
            abort(404);
        }

        $file = File::get($path);
        $type = File::mimeType($path);

        $response = Response::make($file, 200);
        $response->header("Content-Type", $type);

        return $response;
    }


    public function store(Request $request)
    {
        $file = $request->file('file');
        $filename = $file->getClientOriginalName();

        $path = hash( 'sha256', time());
        $local = $path.'/'.$filename;

        if(Storage::disk('uploads')->put($local,  $file)) {
            $fileUpload = new FileUpload();
            $fileUpload->name = $filename;
            $fileUpload->mime = $file->getClientMimeType();
            $fileUpload->path = $local;
            $fileUpload->size = $file->getClientSize();
            $fileUpload->save();

            return response()->json([
                'success' => true,
            ], 200);
        }
        return response()->json([
            'success' => false
        ], 500);
    }


    public function destroy($id)
    {
        $file =  FileUpload::find($id);
        FileUpload::find($id)->delete();
        $path = storage_path('files/uploads/'.$search->path.'/'.$search->name);
        if (file_exists($path)) {
            unlink($path);
            return response()->json([
                'success' => true,
            ], 200);
        }
        return response()->json([
            'success' => false
        ], 500);

    }
}
```

**Configuração do Vue**

Caso não tenha iniciado o Vue dê os comados:

```
npm install
```

**Obs: o Vue precisa ter uma tag com id="app" ou outra definida dentro de resources/assets/js/app.js no objeto Vue em el: '#app'**

Na pasta resources/assets/js clone esse repositório:

```
git clone https://github.com/EmersonBraun/fileupload-laravel-vue.git
```

Registre o componente em resources/assets/js/app.js


```
Vue.component('file-upload', require('./fileupload-laravel-vue/FileUpload.vue'));
```

Agora só chamar em qualquer view assim:

```
<file-upload 
    action="fileupload"
    :ext="['png','jpeg','jpg']" 
    >
</file-upload>
```


Baseado em:  
https://medium.com/@arthursorriso/upload-de-arquivos-com-laravel-e-vuejs-9317cc0097c4
https://appdividend.com/2018/02/13/vue-js-laravel-file-upload-tutorial/
