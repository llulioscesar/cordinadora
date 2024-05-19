# cordinadora Preguntas Técnicas

## 1. Describe cómo implementarías una API REST utilizando Nest.js que interactúe con una base de datos MongoDB. Incluye detalles sobre la estructura de rutas, controladores y servicios.

# Implementación de una API REST con Nest.js y MongoDB

### Paso 1: Crear un nuevo proyecto Nest.js

```bash
npm i -g @nestjs/cli
nest new my-nest-project
```

### PPaso 2: Configurar MongoDB
Primero, instala los paquetes necesarios:
```bash
npm install --save @nestjs/mongoose mongoose
```
Luego, configura MongoDB en el archivo app.module.ts:
```ts
import { Module } from '@nestjs/common';
import { MongooseModule } from '@nestjs/mongoose';
import { UsersModule } from './users/users.module';

@Module({
  imports: [
    MongooseModule.forRoot('mongodb://localhost/nest'), // Conexión a MongoDB
    UsersModule,
  ],
})
export class AppModule {}
```
### Paso 3: Definir módulos, controladores y servicios
Genera un módulo, controlador y servicio para la entidad User:
```bash
nest generate module users
nest generate controller users
nest generate service users
``` 
### Paso 4: Implementar el esquema y modelo de MongoDB
Crea un archivo schemas/user.schema.ts en el módulo de usuarios:
```ts
import { Schema, Prop, SchemaFactory } from '@nestjs/mongoose';
import { Document } from 'mongoose';

@Schema()
export class User extends Document {
  @Prop({ required: true })
  name: string;

  @Prop({ required: true })
  email: string;

  @Prop()
  age: number;
}

export const UserSchema = SchemaFactory.createForClass(User);
```
En el archivo users.module.ts, importa y define el esquema:
```ts
import { Module } from '@nestjs/common';
import { MongooseModule } from '@nestjs/mongoose';
import { UsersService } from './users.service';
import { UsersController } from './users.controller';
import { User, UserSchema } from './schemas/user.schema';

@Module({
  imports: [MongooseModule.forFeature([{ name: User.name, schema: UserSchema }])],
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}
```
### Paso 5: Definir las rutas y los controladores
En el archivo users.controller.ts, define las rutas para manejar las operaciones CRUD:
```ts
import { Controller, Get, Post, Body, Param, Delete, Put } from '@nestjs/common';
import { UsersService } from './users.service';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';

@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Post()
  create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }

  @Get()
  findAll() {
    return this.usersService.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.usersService.findOne(id);
  }

  @Put(':id')
  update(@Param('id') id: string, @Body() updateUserDto: UpdateUserDto) {
    return this.usersService.update(id, updateUserDto);
  }

  @Delete(':id')
  remove(@Param('id') id: string) {
    return this.usersService.remove(id);
  }
}
```
### Paso 6: Implementar los servicios para la lógica de negocio
En el archivo users.service.ts, implementa los métodos para interactuar con la base de datos:

```ts
import { Injectable } from '@nestjs/common';
import { InjectModel } from '@nestjs/mongoose';
import { Model } from 'mongoose';
import { User } from './schemas/user.schema';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';

@Injectable()
export class UsersService {
  constructor(@InjectModel(User.name) private userModel: Model<User>) {}

  async create(createUserDto: CreateUserDto): Promise<User> {
    const createdUser = new this.userModel(createUserDto);
    return createdUser.save();
  }

  async findAll(): Promise<User[]> {
    return this.userModel.find().exec();
  }

  async findOne(id: string): Promise<User> {
    return this.userModel.findById(id).exec();
  }

  async update(id: string, updateUserDto: UpdateUserDto): Promise<User> {
    return this.userModel.findByIdAndUpdate(id, updateUserDto, { new: true }).exec();
  }

  async remove(id: string): Promise<User> {
    return this.userModel.findByIdAndRemove(id).exec();
  }
}
```
### Definir DTOs
Crea los archivos create-user.dto.ts y update-user.dto.ts en el directorio dto:
`create-user.dto.ts`
```ts
export class CreateUserDto {
  readonly name: string;
  readonly email: string;
  readonly age?: number;
}

`update-user.dto.ts`
```ts
import { PartialType } from '@nestjs/mapped-types';
import { CreateUserDto } from './create-user.dto';

export class UpdateUserDto extends PartialType(CreateUserDto) {}
```
### Ejecución de la API
Para ejecutar la API, usa el comando:
```bash
npm run start
```





