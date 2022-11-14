

## MVC design pattern

It helps to enforce separation of concerns to help to avoid mixing presentation logic, business logic, and data access logic altogether.

- Model managed the behavior and data.
- View: manages data display.
- Controller: handles page events and navigation between pages. 

A small example:

1. an user access a specific url with orders?date=today
2. Router interprets said url.
3. Router sends said information to the Controller, which interprets it: give me all orders for today: Orders oList = Store.getOrders(today) 
4. The Controller creates a list of all orders using the Model through C# and accessing the Store and DB.
5. DB receives a query: SELECT * FROM orders WHERE DATE = ´today´, and sends a response to the Model.
6. Model sends the response to the Controller as a list of data.
7. Controller sends said data to a View: Get View("showOrders.html", oList). This generates a list in html format.
8. The list in html format is sent to the browser. 


## Creating a View:

A View will have:
- HTML CSS code or similar.
- Usually gets a list of data from the Controller.
- Dynamically combines data with HTML in a template.
- Uses a language called Razon (ASP.NET). 

The page comes with a Model:
- Data Related.
- Consist of classes / objects with properties.
- Uses SQL statements.
- Suplies the Controller with list of objects. 


Creating the first Model:

```cs
using System;
namespace first_project_jokes.Models
{
	public class Joke
	{
		public int Id { get; set; }
		// getter and setter

		public String JokeQuestion { get; set; }
		public String JokeAnswer { get; set;  }

		// ctor as shortcut to open the constructor
		public Joke()
		{
			// empty because will be used by other pieces of the code
		}
	}
}
``

## Creating a Controller:

Right Click on the Controllers folder: Add new Scaffolding, then configure to include ApplicationDbContext.cs.
Naming convention requires having the word controllers in any Controller file, so JokesController.cs is the name.

The results are:

```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Rendering;
using Microsoft.EntityFrameworkCore;
using first_project_jokes.Data;
using first_project_jokes.Models;

namespace first_project_jokes.Controllers
{
    public class JokesController : Controller
    {
        private readonly ApplicationDbContext _context;

        public JokesController(ApplicationDbContext context)
        {
            _context = context;
        }

        // GET: Jokes
        public async Task<IActionResult> Index()
        {
              return _context.Joke != null ? 
                          View(await _context.Joke.ToListAsync()) :
                          Problem("Entity set 'ApplicationDbContext.Joke'  is null.");
        }

        // GET: Jokes/Details/5
        public async Task<IActionResult> Details(int? id)
        {
            if (id == null || _context.Joke == null)
            {
                return NotFound();
            }

            var joke = await _context.Joke
                .FirstOrDefaultAsync(m => m.Id == id);
            if (joke == null)
            {
                return NotFound();
            }

            return View(joke);
        }

        // GET: Jokes/Create
        public IActionResult Create()
        {
            return View();
        }

        // POST: Jokes/Create
        // To protect from overposting attacks, enable the specific properties you want to bind to.
        // For more details, see http://go.microsoft.com/fwlink/?LinkId=317598.
        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> Create([Bind("Id,JokeQuestion,JokeAnswer")] Joke joke)
        {
            if (ModelState.IsValid)
            {
                _context.Add(joke);
                await _context.SaveChangesAsync();
                return RedirectToAction(nameof(Index));
            }
            return View(joke);
        }

        // GET: Jokes/Edit/5
        public async Task<IActionResult> Edit(int? id)
        {
            if (id == null || _context.Joke == null)
            {
                return NotFound();
            }

            var joke = await _context.Joke.FindAsync(id);
            if (joke == null)
            {
                return NotFound();
            }
            return View(joke);
        }

        // POST: Jokes/Edit/5
        // To protect from overposting attacks, enable the specific properties you want to bind to.
        // For more details, see http://go.microsoft.com/fwlink/?LinkId=317598.
        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> Edit(int id, [Bind("Id,JokeQuestion,JokeAnswer")] Joke joke)
        {
            if (id != joke.Id)
            {
                return NotFound();
            }

            if (ModelState.IsValid)
            {
                try
                {
                    _context.Update(joke);
                    await _context.SaveChangesAsync();
                }
                catch (DbUpdateConcurrencyException)
                {
                    if (!JokeExists(joke.Id))
                    {
                        return NotFound();
                    }
                    else
                    {
                        throw;
                    }
                }
                return RedirectToAction(nameof(Index));
            }
            return View(joke);
        }

        // GET: Jokes/Delete/5
        public async Task<IActionResult> Delete(int? id)
        {
            if (id == null || _context.Joke == null)
            {
                return NotFound();
            }

            var joke = await _context.Joke
                .FirstOrDefaultAsync(m => m.Id == id);
            if (joke == null)
            {
                return NotFound();
            }

            return View(joke);
        }

        // POST: Jokes/Delete/5
        [HttpPost, ActionName("Delete")]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> DeleteConfirmed(int id)
        {
            if (_context.Joke == null)
            {
                return Problem("Entity set 'ApplicationDbContext.Joke'  is null.");
            }
            var joke = await _context.Joke.FindAsync(id);
            if (joke != null)
            {
                _context.Joke.Remove(joke);
            }
            
            await _context.SaveChangesAsync();
            return RedirectToAction(nameof(Index));
        }

        private bool JokeExists(int id)
        {
          return (_context.Joke?.Any(e => e.Id == id)).GetValueOrDefault();
        }
    }
}

```

This also generates a series of Views in a new Jokes folder. Accessing https://localhost:7052/jokes shows this message:

```
A database operation failed while processing the request.
SqliteException: SQLite Error 1: 'no such table: Joke'.
There are pending model changes
Pending model changes are detected in the following:

ApplicationDbContext
In Visual Studio, use the Package Manager Console to scaffold a new migration for these changes and apply them to the database:

PM> Add-Migration [migration name]
PM> Update-Database
Alternatively, you can scaffold a new migration and apply it from a command prompt at your project directory:

> dotnet ef migrations add [migration name]
> dotnet ef database update
```

What to do now?


## Model and DB Creation via Migrations:

Data - Migrations shows db code.

There are schemas for Roles and Users, but not for Jokes. Using Windows the command would be:


```sh
add-migration initialsetup
```

Results are these: 

```cs
using Microsoft.EntityFrameworkCore.Migrations;

#nullable disable

namespace third.Data.Migrations
{
    /// <inheritdoc />
    public partial class initialsetup : Migration
    {
        /// <inheritdoc />
        protected override void Up(MigrationBuilder migrationBuilder)
        {
            migrationBuilder.CreateTable(
                name: "Joke",
                columns: table => new
                {
                    Id = table.Column<int>(type: "int", nullable: false)
                        .Annotation("SqlServer:Identity", "1, 1"),
                    JokeQuestion = table.Column<string>(type: "nvarchar(max)", nullable: false),
                    JokeAnswer = table.Column<string>(type: "nvarchar(max)", nullable: false)
                },
                constraints: table =>
                {
                    table.PrimaryKey("PK_Joke", x => x.Id);
                });
        }

        /// <inheritdoc />
        protected override void Down(MigrationBuilder migrationBuilder)
        {
            migrationBuilder.DropTable(
                name: "Joke");
        }
    }
}

```

This begins a process of Object Relation Mapper. Having a Class and a Table, the ORM creates a table that matchs that class.

DB SETUP, two routes:

DATA ACCESS OBJECT:
- Manually create tables.
- Traditional method of db access.
- Write your own SQL statements.
- DB managers (DBA's) usually prefer this.
- Provides more visibility on finding problems. 

OBJECT RELATIONAL MAPPER:
- Allow computer to generate db tables based on classes defined in app.
- DB is updated using migrations.
- Entity Framework is Microsoft's ORM.
- For basic apps. 

```sh
update-database
```

This will create tables inside one of the SQL Servers, viewable at the SQL Server Object Explorer. 
Also, all [Jokes routes](https://localhost:44300/Jokes) are accessible now.

CRUD functions are available. Just like that. 


## MODIFYING THE LAYOUT

In Views - Shared - _Layout: 

```html
<li class="nav-item">
    <a class="nav-link text-dark" asp-area="" asp-controller="Home" asp-action="Privacy">Privacy</a>
</li>

<li class="nav-item">
    <a class="nav-link text-dark" asp-area="" asp-controller="Jokes" asp-action="Index">Jokes</a>
</li>
```

Why Jokes and Index? What is asp-controller and asp-action? 
That info comes from the Jokes Controller:

```cs
// this will show all Jokes
        public async Task<IActionResult> Index()
        {
              return _context.Joke != null ? 
                          View(await _context.Joke.ToListAsync()) :
                          Problem("Entity set 'ApplicationDbContext.Joke'  is null.");
        }

```

Why not asp-controller='JokesController'? Controller is assumed. It's a naming convention. 


## CREATE A SEARCH JOKES FEATURE

in _Layout:

```html
                        <li class="nav-item">
                            <a class="nav-link text-dark" asp-area="" asp-controller="Jokes" asp-action="ShowSearchForm">Jokes</a>
                        </li>
```

ShowSearchForm does not exist yet. It needs to be created as a method in the JokesController.cs

```cs
        // GET: Jokes/ShowSearchForm

        public async Task<IActionResult> ShowSearchForm()
        {
            return _context.Joke != null ?
                    // since the name is ShowSearchForm, View() can be empty
                        View() :
                        Problem("Entity set 'ApplicationDbContext.Joke'  is null.");
        }
```

By itself, this is not enough. Right click ShowSearchForm and Add View. The result will be this:

```cshtml
@model third.Models.Joke

<h4>Joke</h4>
<hr />
<div class="row">
    <div class="col-md-4">
        <form asp-action="ShowSearchForm">
            <div asp-validation-summary="ModelOnly" class="text-danger"></div>
            <div class="form-group">
                <label asp-for="JokeQuestion" class="control-label"></label>
                <input asp-for="JokeQuestion" class="form-control" />
                <span asp-validation-for="JokeQuestion" class="text-danger"></span>
            </div>
            <div class="form-group">
                <label asp-for="JokeAnswer" class="control-label"></label>
                <input asp-for="JokeAnswer" class="form-control" />
                <span asp-validation-for="JokeAnswer" class="text-danger"></span>
            </div>
            <div class="form-group">
                <input type="submit" value="Create" class="btn btn-primary" />
            </div>
        </form>
    </div>
</div>

<div>
    <a asp-action="Index">Back to List</a>
</div>

@section Scripts {
    @{await Html.RenderPartialAsync("_ValidationScriptsPartial");}
}

``` 

### About the ApplicationDbContext.cs file:

It is important when creating Controllers. 