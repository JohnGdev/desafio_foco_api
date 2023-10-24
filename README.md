

# A API foi criada para um sistema de hotelaria, destinado ao gerenciamento de quartos/acomodações e reservas, utilizando o padrão RESTful.
- Foi uitilizado o Swagger para a domuntação utilizando o padrão OpenApi.
- Utilizei um comando para salvar na pasta api-documentation em um arquivo chamado swagger.json
- Depopis é exibido na index que está na mesma pasta.

![image](https://github.com/JohnGdev/desafio_foco_api/assets/106494649/a17c1212-ec30-4b96-b718-268ab8e80e33)

## Requisitos

- [ ] Versão do PHP 8.2.11.
- [ ] Banco de dados MySQL.
- [ ] Composer instalado.
- [ ] Laravel Framework 10.28.0

comandos utilizados para criar o projeto:
```bash
composer create-project --prefer-dist laravel/laravel DESAFIO-FOCO
````
comandos para criar os models
```bash
php artisan make:model Room -m 
php artisan make:model Guest -m    
php artisan make:model Hotel -m    
php artisan make:model Payment -m
php artisan make:model Reserve -m    
php artisan make:model Room -m    
````
Comando pra criar o controller pra importar os xmls
```bash
php artisan make:controller XmlImportController  
````
Comando pra criar o controller de Quarto e Reserva
php artisan make:controller RoomController
php artisan make:controller ReserveController

foi utilizado o eloquent do laravel para criar as tabelas no banco de dados :

Tabela Hotels

```bash
 public function up(): void
    {
        Schema::create('hotels', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('hotels');
    }
};
````

Tabela Rooms

```bash
 public function up(): void
    {
        Schema::create('rooms', function (Blueprint $table) {
            $table->id();
            $table->unsignedBigInteger('hotel_id');
            $table->string('name');
            $table->timestamps();
            $table->foreign('hotel_id')->references('id')->on('hotels');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('rooms');
    }

````
Tabela Reserves

```bash
 public function up(): void
    {
        Schema::create('reserves', function (Blueprint $table) {
            $table->id();
            $table->unsignedBigInteger('hotel_id');
            $table->unsignedBigInteger('room_id');
            $table->date('checkIn');
            $table->date('checkOut');
            $table->decimal('total', 8, 2);
            $table->timestamps();
            $table->foreign('hotel_id')->references('id')->on('hotels');
            $table->foreign('room_id')->references('id')->on('rooms');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('reserves');
    }
````

Tabela Daily

```bash
  public function up(): void
    {
        Schema::create('dailies', function (Blueprint $table) {
            $table->id();
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('dailies');
    }
````
Tabela Payments
```bash

  public function up(): void
    {
        Schema::create('payments', function (Blueprint $table) {
            $table->unsignedBigInteger('reserve_id');
            $table->integer('method'); // Você pode criar uma tabela de métodos de pagamento separada
            $table->decimal('value', 8, 2);
            $table->timestamps();
            $table->foreign('reserve_id')->references('id')->on('reserves');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('payments');
    }

````
Tabela Guests

```bash


 public function up(): void
    {
        Schema::create('guests', function (Blueprint $table) {
            $table->unsignedBigInteger('reserve_id');
            $table->string('name');
            $table->string('lastName');
            $table->string('phone');
            $table->timestamps();
            $table->foreign('reserve_id')->references('id')->on('reserves');
        });
    }


    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('guests');
    }
````
existem algumas outras migrations que foram para ajustes durante o desenvolvimento 
para rodar as migrations foi utilizado o comado:

```bash
php artisan migrate 
````

Em alguns models foram implentado algumas regras exemplo:
```bash
class Reserve extends Model
{
    protected $fillable = ['hotel_id', 'room_id', 'checkIn', 'checkOut', 'total'];

    static public $rules = [
        'hotel_id'=> 'required|exists:App\Models\Hotel,id',
        'room_id'=> 'required|int',
        'checkIn'=> 'required|date',
        'checkOut'=> 'date|after:checkIn'
    ];


````


Nos controllers foram implementados os metodos da api listarei alguns também foi craido uma documentação no swagger http://127.0.0.1:8000/api-documentation#/ :

```bash
public function list()
    {
        $rooms = Room::all();
        return $rooms->toJson();
    }



public function create(Request $request)
    {
        $request->validate(Room::$rules);

        $room = Room::create($request->all());
        return $room->toJson();
    }


    public function getById($id)
    {
        $room = Room::find($id);
        if (!$room) {
            return response()->json(['message' => 'Room not found'], 404);
        }
        return $room->toJson();
    }


 public function edit(Request $request, $id)
    {
        $room = Room::find($id);

        if (!$room) {
            return response()->json(['message' => 'Room not found'], 404);
        }

        $room->name = is_null($request->name) ? $room->name : $request->name;
        $room->hotel_id = is_null($request->hotel_id) ? $room->hotel_id : $request->hotel_id;

        $room->save();

        return $room->toJson();
    }


     public function destroy($id)
    {
        $room = Room::find($id);
        if (!$room) {
            return response()->json(['message' => 'Room not found'], 404);
        }

        if ($room->reserves()->exists()) {
            return response()->json(['message' => 'Cannot delete the room as it is associated with reservations:'], 400);
        }

        $room->hotel()->dissociate();

        $room->delete();

        return response()->json(['message' => 'Room deleted successfully']);
    }


````

As rotas foram criadas no arquivo api.php :

O padrão é consistente e segue as melhores práticas para definir rotas no Laravel, tornando a API clara e fácil de entender. Cada rota corresponde a uma ação específica em um controlador.

```bash
// Rooms

Route::get('/rooms', 'App\Http\Controllers\RoomController@list');
Route::post('/rooms', 'App\Http\Controllers\RoomController@create');
Route::get('/rooms/{id}', 'App\Http\Controllers\RoomController@getById');
Route::patch('/rooms/{id}', 'App\Http\Controllers\RoomController@edit');
Route::delete('/rooms/{id}', 'App\Http\Controllers\RoomController@destroy');

// Reserves

Route::get('/reserves', 'App\Http\Controllers\ReserveController@list');
Route::post('/reserves', 'App\Http\Controllers\ReserveController@createReserve');
Route::post('/reserves/{reserveId}/dailies', 'App\Http\Controllers\ReserveController@addDailies');
Route::post('/reserves/{reserveId}/payments', 'App\Http\Controllers\ReserveController@addPayments');

````

A impotação dos arquivos xmls se dão atraves de um comando que foi criado na pasta Comands:
```bash

php artisan app:import-xmls 
````
```bash

class ImportXmls extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'app:import-xmls';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Import the XMLs file to update the database';

    /**
     * Execute the console command.
     */
    public function handle()
    {
        try {
            DB::beginTransaction();

            $controller = new XmlImportController();

            $controller->importHotels();
            $controller->importRooms();
            $controller->importReserves();

            echo 'Importação concluída';
            DB::commit();
        } catch (\Throwable $th) {
            DB::rollBack();
            echo 'Ocorreu um erro durante a importação';
            echo $th->getMessage();
        }
    }
}
````
