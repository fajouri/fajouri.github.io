---
title:  "¿Cómo implementar un ABM con Blazor usando Entity Framework Core? Demostración detallada"
categories:
  - Desarrollo
tags:
  - c#
  - Blazor
  - .Net core
  - ABM
  - Crud
---

## ¿Cómo implementar un ABM con Blazor usando Entity Framework Core? Demostración detallada

Tomando como base el ejemplo de [Mukesh Murugan](https://codewithmukesh.com/blog/blazor-crud-with-entity-framework-core/) vamos a describir detalladamente la creacion de un abm con blazor y Entity Framework Core

La creación de una aplicación ABM es como el Hola mundo para desarrolladores intermedios. Ayuda a comprender las operaciones más comunes de cualquier stack de tecnologias en particular. En este tutorial, creamos una aplicación Blazor ABM del lado del cliente que usa Entity Framework Core como su capa de acceso a datos.

Blazor es la novedad del mundo .NET . Es muy importante para un desarrollador estar al tanto de lo que esta increíble tecnología contiene en sí misma. ¿Qué mejor forma de comprender sus habilidades que hacerlo aprendiendo a implementar funciones de ABM utilizando Blazor y Entity Framework Core?

## Qué vamos a construir
Pensé que sería bueno si les mostrara lo que vamos a construir antes de saltar al tutorial para que tengan una buena idea del alcance de este tutorial y lo cubriremos. Tendremos una implementación simple de Blazor ABM en la entidad Programador usando Blazor WebAssembly y Entity Framework Core. También intentaré demostrar ciertas buenas prácticas al desarrollar aplicaciones ABM de Blazor.

![Pantalla que lista los programadores que encuentra en el DB](/assets/images/Listado.PNG)

## Creando el proyecto Blazor ABM
Comencemos con la aplicación Blazor WebAssembly ABM creando una nueva aplicación Blazor. Siga las capturas de pantalla a continuación. 

Puedes ver que Visual Studio genera automáticamente 3 proyectos diferentes, es decir, Cliente, Servidor y Compartido (shared).
1. Cliente: donde reside la aplicación Blazor junto con los componentes de Razor.
2. Servidor: se usa principalmente como un contenedor que tiene características ASP.NET Core (lo usamos aquí para EF Core, controladores Api [Api controllers] y DB).
3. Compartido: como sugiere el nombre, todos los modelos de entidad se definirán aquí.

**Básicamente, el servidor tendrá los puntos de entrada de la API (End Points) a los que puede acceder el proyecto cliente. Tenga en cuenta que esta es una aplicación única y se ejecuta en el mismo puerto. Por lo tanto, no surge la necesidad de acceso a CORS. Todos estos proyectos se ejecutarán directamente en su navegador y no necesitan un servidor dedicado.**

Para comenzar con nuestra aplicación Blazor ABM, agreguemos un modelo de Programador a nuestro proyecto compartido en la carpeta Modelos (crea la carpeta). 
```
   public class Programador
    {
        public int Id { get; set; }
        public string Nombre { get; set; }
        public string Apellido { get; set; }
        public string Email { get; set; }
        public decimal Experiencia { get; set; }
    }
```
## Entity Framework Core
Ahora, vaya al proyecto del servidor e instale los siguientes paquetes necesarios para habilitar EF Core.
```
Install-Package Microsoft.EntityFrameworkCore
Install-Package Microsoft.EntityFrameworkCore.Design
Install-Package Microsoft.EntityFrameworkCore.Tools
Install-Package Microsoft.EntityFrameworkCore.SqlServer
```
Vaya a appsettings.json en el Proyecto de servidor y agregue la Cadena de conexión.
```C#
"ConnectionStrings": {
    "DefaultConnection": "<Connection String Here>"
  },
  
```
## Agregar contexto de aplicación
Necesitaremos un contexto de base de datos para trabajar con los datos. Cree una nueva clase en el proyecto de servidor en Data / AplicacionDBContext.cs
```
 public class AplicacionDbContext:DbContext
 {
   public AplicacionDbContext(DbContextOptions<AplicacionDbContext> options) : base(options)
   {
   }
 
 public DbSet<Programador> Programadores { get; set;}
```
Agregaremos DbSet de Programadores a nuestro contexto. Usando esta clase de contexto, podremos realizar operaciones en nuestra base de datos que generaremos mas adelante.

# Configuración
Necesitaremos agregar EF Core y definir su cadena de conexión. Navegue hasta Startup.cs que se encuentra en el proyecto del servidor y agregue la siguiente línea al método ConfigureServices.
```
services.AddDbContext<AplicacionDbContext>(op =>
                op.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));
```
## Generación / migración de la base de datos
Ahora que nuestro Entity Framework Core está configurado y listo en la aplicación ASP.NET Core, hagamos las migraciones y actualicemos nuestra base de datos. Para esto, abra la Consola del Administrador de paquetes y escriba lo siguiente.
```
add-migration Initial
update-database
```
### Importante: asegúrese de haber elegido el proyecto del servidor como predeterminado.

## Controller de Programadores.
Ahora que nuestra base de datos y EF Core están configurados, crearemos un controlador API que entregará los datos a nuestro cliente Blazor. Cree un controlador API vacío, Controllers/ProgramadorController.cs

```C#
[Route("api/[controller]")]
[ApiController]
public class ProgramadorController : ControllerBase
{
   private readonly AplicacionDbContext _context;
   public ProgramadorController(AplicacionDbContext context)
   {
    _context = context;
   }
}
```

Aquí hemos inyectado una nueva instancia de ApplicationDBContent al constructor del controlador. Continuemos agregando cada uno de los end points o metodos para nuestras operaciones de ABM.

## GET
Un método para obtener todos los programadores de la instancia de contexto.
```
[HttpGet]
public async Task<IActionResult> Get()
{
   var programadores = await _context.Programadores.ToListAsync();
   return Ok(programadores);
}
```

## GET BY ID
Obtener por identificación el detalles de un desarrollador que coincide con el ID pasado como parámetro. 

```
[HttpGet("{id}")]
public async Task<IActionResult> Get(int id)
{
  var programador = await _context.Programadores.FirstOrDefaultAsync(a => a.Id == id);
   return Ok(programador);
}
```
## Create
Creamos un nuevo Programador con el objeto pasado como parametro.
```
[HttpPost]
public async Task<IActionResult> Post(Programador programador)
{
   _context.Add(programador);
   await _context.SaveChangesAsync();
   return Ok(programador.Id);
}
```

## Update
Modificamos un programador existente.
```
[HttpPut]
public async Task<IActionResult> Put(Programador programador)
{
   _context.Entry(programador).State = EntityState.Modified;
   await _context.SaveChangesAsync();
   return NoContent();
}
```

## Delete
Eliminamos un programador por su Id.

```
[HttpDelete("{id}")]
public async Task<IActionResult> Delete(int id)
{
   var programador = new Programador { Id = id };
   _context.Remove(programador);
   await _context.SaveChangesAsync();
   return NoContent();
}
```
# Introducción al ABM en Blazor
Una vez terminado el Backend, continuemos construyendo nuestra aplicación ABM Blazor. Nuestra Agenda es hacer Operaciones de ABM (CRUD en ingles) en la Entidad Programador. Básicamente, hemos completado nuestra capa de datos. Construyamos ahora la interfaz de usuario.

## Agregar una nueva entrada al menú de navegación
Tendremos que agregar una nueva entrada en la barra lateral del menú de navegación para acceder a las Páginas. En el proyecto del cliente, navegue a Shared/NavMenu.razor y agregue una entrada similar a la siguiente.

```C#
<li class="nav-item px-3">
   <NavLink class="nav-link" href="programador">
      <span class="oi oi-people" aria-hidden="true"></span> Programadores
   </NavLink>
 </li>
```

# Espacios de nombres compartidos
A lo largo de este tutorial usaremos la clase de modelo Shared/Models/Programador.cs. Por lo tanto agreguemos este espacio de nombres a _Imports.razor, para que se pueda acceder a él en todos nuestros componentes nuevos.
```
@using Blazor.Abm.Shared.Models
```
# Estructura de carpeta sugerida
Esto es considerado una buena práctica. La idea es la siguiente.

Tenga una carpeta debajo de las páginas que serán específicas para una entidad. En nuestro caso, tenemos un Programador como entidad.
En esta carpeta, intentamos incluir todos los componentes de Razor en cuestión.

![Estructura de carpeta sugerida](/assets/images/Carpeta.PNG)


Si ya es un desarrollador de ASP.NET Core, sabrá que mientras implementamos un ABM estándar, tenemos muchas cosas similares (UI) para crear y actualizar formularios. Para ello, creamos un componente común llamado Formulario que será compartido por los componentes Crear y Editar.
Listado.razor es la pantalla principal que permitr recuperar todos los datos y mostrarlos en una tabla Bootstrap.

# Componente Listado
Comenzaremos construyendo nuestro componente de índice que busca a todos los programadores de la base de datos. Llamémoslo Componente Listado. Cree un nuevo componente Blazor en Pages, Pages/Programadores/Listado.razor

```
@page "/programador"
@inject HttpClient clienteHttp
@inject IJSRuntime js
<h3>Programadores</h3>
<small>Agrega los programadores que desee</small>
<div class="form-group">
    <a class="btn btn-success" href="programador/create"><i class="oi oi-plus"></i> Nuevo</a>
</div>
<br>
@if (programadores == null)
{
    <text>Loading...</text>
}
else if (!programadores.Any())
{
    <text>No hay datos.</text>
}
else
{
    <table class="table table-striped">
        <thead>
        <tr>
            <th>Id</th>
            <th>Nombre</th>
            <th>Apellido</th>
            <th>Email</th>
            <th>Experiencia (Años)</th>
            <th></th>
        </tr>
        </thead>
        <tbody>
        @foreach (var programador in programadores)
        {
            <tr>
                <td>@programador.Id</td>
                <td>@programador.Nombre</td>
                <td>@programador.Apellido</td>
                <td>@programador.Email</td>
                <td>@programador.Experiencia</td>
                <td>
                    <a class="btn btn-success" href="programador/edit/@programador.Id">Editar</a>
                    <button class="btn btn-danger" @onclick="@(() => Delete(programador.Id))">Eliminar</button>
                </td>
            </tr>
        }
        </tbody>
    </table>
}
@code {
    Programador[] programadores { get; set; }

    protected override async Task OnInitializedAsync()
    {
        programadores = await clienteHttp.GetFromJsonAsync<Programador[]>("api/programador");
    }

    async Task Delete(int id)
    {
        var programador = programadores.First(x => x.Id == id);
        if (await js.InvokeAsync<bool>("confirm", $"Quiere eliminar el registro de {programador.Nombre} ({programador.Id})?"))
        {
            await clienteHttp.DeleteAsync($"api/programador/{id}");
            await OnInitializedAsync();
        }
    }

}
```

## ¿Qué es IJSRuntime?
Anteriormente, hemos aprendido que también es posible ejecutar Javascripts en nuestra aplicación Blazor. La interfaz IJSRuntime actúa como puerta de entrada a JS. Este objeto nos proporciona métodos para invocar funciones JS en aplicaciones Blazor.

Creemos nuestra aplicación! y hagamos la prueba.


¡Tu primer componente ABM esta listo! 😀 Puedes ver que no tenemos ningún registro. Agreguemos un componente Create Razor para poder agregar nuevas entradas a la base de datos.

### Pero antes, necesitaremos crear el formilario comun que compartiremos entre los componentes de cración y edición.

## Componente Formulario
Agreguemos un nuevo componente Razor en Pages/Programadores/Formulario.razor

```
<EditForm Model="@programador" OnValidSubmit="@OnValidSubmit">
    <DataAnnotationsValidator />
    <div class="form-group">
        <label>Nombre :</label>
        <div>
            <InputText @bind-Value="@programador.Nombre" />
            <ValidationMessage For="@(() => programador.Nombre)" />
        </div>
    </div>
    <div class="form-group ">
        <div>
            <label>Apellido :</label>
            <div>
                <InputText @bind-Value="@programador.Apellido" />
                <ValidationMessage For="@(() => programador.Apellido)" />
            </div>
        </div>
    </div>
    <div class="form-group ">
        <div>
            <label>Email :</label>
            <div>
                <InputText @bind-Value="@programador.Email" />
                <ValidationMessage For="@(() => programador.Email)" />
            </div>
        </div>
    </div>
    <div class="form-group ">
        <div>
            <label>Experiencia :</label>
            <div>
                <InputNumber @bind-Value="@programador.Experiencia" />
                <ValidationMessage For="@(() => programador.Experiencia)" />
            </div>
        </div>
    </div>
    <button type="submit" class="btn btn-success">
        @ButtonText
    </button>
</EditForm>
@code {
    [Parameter] public Programador programador { get; set; }
    [Parameter] public string ButtonText { get; set; } = "Guardar";
    [Parameter] public EventCallback OnValidSubmit { get; set; }
}
```

_Pensando en esto, siento que esto es bastante similar a los conceptos de Interfaz y Concreto. Entiéndalo de esta manera. El componente Formulario es la interfaz que tiene las propiedades y métodos necesarios. Y el componente Crear / Editar sería la clase Concreto que tiene la implementación real de las propiedades / métodos de la interfaz. ¿No tiene sentido? Lea este párrafo nuevamente después de pasar por la sección Crear componente._

## ¿Para qué sirve la etiqueta de parámetro?
En Blazor, se puede agregar parámetros a cualquier componente decorándolos con una etiqueta de parámetro [Parameter]. Hace disponible para que los componentes externos pasen estos parámetros. En nuestro caso, hemos definido el objeto Programador como uno de los parámetros de los componentes del Formulario. Por lo tanto, los otros componentes que utilizarán los componentes de formulario, es decir, los componentes de creación / edición tienen una opción para pasar el objeto de programador como un parámetro a los componentes de formulario. Esto ayuda a una alta reutilización de los componentes.
```
@code {
    [Parameter] public Programador programador { get; set; }
    [Parameter] public string ButtonText { get; set; } = "Guardar";
    [Parameter] public EventCallback OnValidSubmit { get; set; }
}
```

## Componente Crear  
Ahora, comencemos a utilizar el componente Formulario creado anteriormente. Construiremos una interfaz para agregar un nuevo programador. Cree un nuevo componente Razor, Pages/Programadores/Create.razor
```
@page "/programador/create"
@inject HttpClient ClienteHttp
@inject NavigationManager UriHelper
<h3>Alta</h3>
<Formulario ButtonText="Nuevo Programador" programador="@programador" OnValidSubmit="@NuevoProgramador" />

@code {
    Programador programador = new Programador();
    async Task NuevoProgramador()
    {
        await ClienteHttp.PostAsJsonAsync("api/programador", programador);
        UriHelper.NavigateTo("programador");
    }
}
```

Ejecute la aplicación para hacer una prueba. Vaya a la pestaña Programadores y haga clic en el botón Nuevo. Aquí agregue datos de muestra y presione Nuevo Programador.

![Pantalla para crear programadores](/assets/images/Crear.PNG)

### Te sorprenderá la velocidad de la aplicación. Bastante suave, ¿no? Además, ya hemos construido bastante bien nuestro componente Crear. Ahora construyamos nuestro componente final. Actualizar.

# Componente Editar
Cree un nuevo componente en Pages/Programadores/Edit.razor
```
@page "/programador/edit/{programadorId:int}"
@inject HttpClient Clientehttp
@inject NavigationManager uriHelper
@inject IJSRuntime js
<h3>Editar</h3>
<Formulario ButtonText="Actualizar" programador="@programador"
      OnValidSubmit="@EditarProgramador" />

@code {
    [Parameter] public int programadorId { get; set; }
    Programador programador = new Programador();
    protected async override Task OnParametersSetAsync()
    {
        programador = await Clientehttp.GetFromJsonAsync<Programador>($"api/programador/{programadorId}");
    }
    async Task EditarProgramador()
    {
        await Clientehttp.PutAsJsonAsync("api/programador", programador);
        await js.InvokeVoidAsync("alert", $"Se ha modificado exitosamente!");
        uriHelper.NavigateTo("programador");
    }
}
```
![Pantalla para editar programadores](/assets/images/Editar.PNG)
![Actualizacion](/assets/images/Actualizado.PNG)

Resumen de la aplicación ABM con Blazor 
A estas alturas, podemos crear una aplicación completa ABM con Blazor desde cero. Hemos cubierto los conceptos básicos de ABM con Blazor, sus etiquetas y la reutilización de componentes. También hemos visto una estructura de carpetas considerada como buena práctica que es bastante aplicable para las aplicaciones ABM Blazor en general. Espero que hayan entendido y disfrutado este tutorial detallado. Puede encontrar el código fuente completo [aquí](https://github.com/fajouri/Blazor-ABM-Core-5-EF-).
