# cordinadora Preguntas Técnicas

## 1. Describe cómo implementarías una API REST utilizando Nest.js que interactúe con una base de datos MongoDB. Incluye detalles sobre la estructura de rutas, controladores y servicios.

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
import {Module} from '@nestjs/common';
import {MongooseModule} from '@nestjs/mongoose';
import {UsersModule} from './users/users.module';

@Module({
    imports: [
        MongooseModule.forRoot('mongodb://localhost/nest'), // Conexión a MongoDB
        UsersModule,
    ],
})
export class AppModule {
}
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
import {Schema, Prop, SchemaFactory} from '@nestjs/mongoose';
import {Document} from 'mongoose';

@Schema()
export class User extends Document {
    @Prop({required: true})
    name: string;

    @Prop({required: true})
    email: string;

    @Prop()
    age: number;
}

export const UserSchema = SchemaFactory.createForClass(User);
```

En el archivo users.module.ts, importa y define el esquema:

```ts
import {Module} from '@nestjs/common';
import {MongooseModule} from '@nestjs/mongoose';
import {UsersService} from './users.service';
import {UsersController} from './users.controller';
import {User, UserSchema} from './schemas/user.schema';

@Module({
    imports: [MongooseModule.forFeature([{name: User.name, schema: UserSchema}])],
    controllers: [UsersController],
    providers: [UsersService],
})
export class UsersModule {
}
```

### Paso 5: Definir las rutas y los controladores

En el archivo users.controller.ts, define las rutas para manejar las operaciones CRUD:

```ts
import {Controller, Get, Post, Body, Param, Delete, Put} from '@nestjs/common';
import {UsersService} from './users.service';
import {CreateUserDto} from './dto/create-user.dto';
import {UpdateUserDto} from './dto/update-user.dto';

@Controller('users')
export class UsersController {
    constructor(private readonly usersService: UsersService) {
    }

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
import {Injectable} from '@nestjs/common';
import {InjectModel} from '@nestjs/mongoose';
import {Model} from 'mongoose';
import {User} from './schemas/user.schema';
import {CreateUserDto} from './dto/create-user.dto';
import {UpdateUserDto} from './dto/update-user.dto';

@Injectable()
export class UsersService {
    constructor(@InjectModel(User.name) private userModel: Model<User>) {
    }

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
        return this.userModel.findByIdAndUpdate(id, updateUserDto, {new: true}).exec();
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
```

`update-user.dto.ts`

```ts
import {PartialType} from '@nestjs/mapped-types';
import {CreateUserDto} from './create-user.dto';

export class UpdateUserDto extends PartialType(CreateUserDto) {
}
```

### Ejecución de la API

Para ejecutar la API, usa el comando:

```bash
npm run start
```

# Explica la diferencia entre Server Side Rendering y Static Site Generation en Next.js. ¿Cuándo utilizarías uno sobre el otro?

`Server Side Rendering (SSR)` y `Static Site Generation (SSG)` son dos métodos de generación de páginas disponibles en
Next.js, cada uno con sus propias características y casos de uso. A continuación, se detallan las diferencias y se
explica cuándo utilizar cada uno.

### Server Side Rendering (SSR)

SSR se refiere a la generación de páginas dinámicamente en el servidor en cada solicitud. Next.js proporciona SSR a
través de la función getServerSideProps.

Características:

• Generación Dinámica: Las páginas se generan en el servidor cada vez que se recibe una solicitud.
• Datos Actualizados: Es ideal para páginas que necesitan mostrar datos actualizados en cada solicitud.
• Menor Tiempo de Carga Inicial: La primera carga puede ser más lenta debido a la necesidad de generar la página en cada
solicitud.

Uso de SSR:

• Aplicaciones de Comercio Electrónico: Donde los precios y la disponibilidad de los productos pueden cambiar con
frecuencia.
• Dashboards y Paneles de Control: Que requieren datos actualizados en tiempo real.
• Contenido Personalizado: Páginas que muestran contenido personalizado para los usuarios, como datos de usuario
autenticado.

### Static Site Generation (SSG)

SSG se refiere a la generación de páginas estáticas en el momento de la construcción (build time). Next.js proporciona
SSG a través de la función getStaticProps.

Características:

- Generación Estática: Las páginas se generan una vez durante el proceso de construcción y se sirven como HTML estático.
- Rendimiento Optimo: Las páginas cargan rápidamente porque no necesitan ser generadas en cada solicitud.
- Actualización Incremental: Puede configurarse para regenerar páginas estáticas periódicamente o cuando se reciben
  nuevas solicitudes (Incremental Static Regeneration, ISR).

Uso de SSG:

- Blogs y Sitios de Documentación: Donde el contenido no cambia con frecuencia.
- Páginas de Marketing y Landing Pages: Que requieren tiempos de carga rápidos para mejorar la experiencia del usuario.
- Contenido Predeterminado: Páginas que muestran datos que no cambian frecuentemente o que pueden ser prerenderizadas de
  antemano.

### ¿Cuándo utilizar uno sobre el otro?

Usa SSR cuando:

- Necesites datos actualizados en cada solicitud.
- El contenido cambia frecuentemente y debe reflejarse inmediatamente.
- Requieras personalización basada en la solicitud del usuario o en cookies.

Usa SSG cuando:

- El contenido es estático o cambia infrecuentemente.
- Deseas tiempos de carga rápidos y un rendimiento óptimo.
- Puedes prerenderizar el contenido durante el build y no necesitas datos en tiempo real.

# Compara el manejo de la autenticación en una aplicación .NET Core con una aplicación PHP. ¿Qué herramientas utilizarías en cada caso?

El manejo de la autenticación en aplicaciones .NET Core y PHP puede ser diferente en términos de herramientas,
implementación y buenas prácticas. A continuación, se presenta una comparación detallada entre ambas plataformas y las
herramientas comúnmente utilizadas en cada caso.

### Autenticación en .NET Core

`.NET Core` ofrece un conjunto robusto de herramientas y bibliotecas para la autenticación, que están integradas en el
marco. Aquí se detallan algunos aspectos clave y herramientas utilizadas:

Herramientas y Bibliotecas Comunes:

1. ASP.NET Core Identity:

- Descripción: Es una biblioteca completa que proporciona funcionalidades para la gestión de usuarios, roles y
  autenticación.
- Características: Registro de usuarios, recuperación de contraseñas, autenticación de dos factores, integración con
  proveedores externos como Google, Facebook, etc.
- Uso:
  ```csharp
  services.AddIdentity<ApplicationUser, IdentityRole>()
        .AddEntityFrameworkStores<ApplicationDbContext>()
        .AddDefaultTokenProviders();
  ```

2. JWT (JSON Web Tokens):
    - Descripción: Es utilizado para la autenticación basada en tokens. Es ideal para aplicaciones RESTful.
    - Características: Tokens seguros y firmados que pueden ser utilizados para validar la identidad de los usuarios.
    - Uso:
       ```csharp
       services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
            .AddJwtBearer(options =>
            {
                options.TokenValidationParameters = new TokenValidationParameters
                {
                    ValidateIssuer = true,
                    ValidateAudience = true,
                    ValidateLifetime = true,
                    ValidateIssuerSigningKey = true,
                    ValidIssuer = "yourissuer.com",
                    ValidAudience = "youraudience.com",
                    IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("your_secret_key"))
                };
            });
       ```

3. OAuth/OpenID Connect:
    - Descripción: Protocolos estándar para la autenticación y autorización.
    - Características: Integración con proveedores de identidad externos como Azure AD, Google, y otros.
    - Uso:
       ```csharp
      services.AddAuthentication(options =>
      {
         options.DefaultAuthenticateScheme = CookieAuthenticationDefaults.AuthenticationScheme;
         options.DefaultChallengeScheme = OpenIdConnectDefaults.AuthenticationScheme;
      })
      .AddCookie()
      .AddOpenIdConnect(options =>
      {
          options.ClientId = "client_id";
          options.ClientSecret = "client_secret";
          options.Authority = "https://your-identity-provider.com";
      });
      ```

### Autenticación en PHP

En PHP, el manejo de la autenticación puede variar dependiendo del framework utilizado. Las herramientas y bibliotecas
no están tan integradas como en .NET Core, pero hay opciones robustas disponibles.

Herramientas y Bibliotecas Comunes:

1. Laravel Authentication:
   - Descripción: Laravel, uno de los frameworks más populares de PHP, ofrece un sistema de autenticación completo.
   - Características: Gestión de usuarios, roles, políticas, autenticación de dos factores, etc.
   - Uso:
     ```php
     // En Laravel, se puede generar el sistema de autenticación con un solo comando
      php artisan make:auth
     ```

2. JWT (JSON Web Tokens):

   - Descripción: Similar a .NET Core, JWT se utiliza para la autenticación basada en tokens.
   - Características: Tokens seguros y firmados.
   - Biblioteca Recomendada: firebase/php-jwt
   - Uso:
        ```php
        use \Firebase\JWT\JWT;
    
        $token = array(
            "iss" => "http://example.org",
            "aud" => "http://example.com",
            "iat" => 1356999524,
            "nbf" => 1357000000
        );
    
        $jwt = JWT::encode($token, $key);
        ```
3. OAuth/OpenID Connect:
   - Descripción: Protocolos estándar para autenticación y autorización.
   - Biblioteca Recomendada: league/oauth2-client
   - Uso:
      ```php
     $provider = new \League\OAuth2\Client\Provider\GenericProvider([
        'clientId'                => 'demoapp',    // The client ID assigned to you by the provider
        'clientSecret'            => 'demopass',   // The client password assigned to you by the provider
        'redirectUri'             => 'http://example.com/your-redirect-url/',
        'urlAuthorize'            => 'https://example.com/oauth2/authorize',
        'urlAccessToken'          => 'https://example.com/oauth2/token',
        'urlResourceOwnerDetails' => 'https://example.com/oauth2/resource'
        ]);
     ```
### Comparación y Casos de Uso

.NET Core:

 - Ventajas:
 - Herramientas integradas y robustas.
 - Soporte completo para estándares de seguridad.
 - Integración fluida con otros servicios de Microsoft.
 - Cuándo usar:
 - Aplicaciones empresariales.
 - Aplicaciones que requieren alta seguridad y características avanzadas de autenticación.
 - Proyectos que ya utilizan el ecosistema de Microsoft.

PHP:

 - Ventajas:
 - Flexibilidad en la elección de herramientas y bibliotecas.
 - Amplio ecosistema de paquetes y soporte de la comunidad.
 - Fácil de desplegar en la mayoría de los servidores web.
 - Cuándo usar:
 - Proyectos pequeños y medianos.
 - Sitios web que necesitan autenticación simple.
 - Aplicaciones que ya utilizan frameworks populares de PHP como Laravel.
