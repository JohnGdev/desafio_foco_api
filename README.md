# Título da API

Descrição concisa da sua API e sua finalidade.

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

