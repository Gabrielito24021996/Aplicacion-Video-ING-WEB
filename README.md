# Aplicacion-Video-ING-WEB usando Core 2.1 MVC

Realizaremos un mini-proyecto para las personas que quieran empezar con este Framework puedan entender de mejor manera su funcionamiento.

Como este es un MVC necesitamos primero crear la Vistas, despues crearemos un Controller y por ultimo crearemos la Vista.

Para el manejo de la base de datos, aqui Microsoft ya con la creacion de los modelos nos crea las tabals de datos con sus respectivas claves primarias y secundarias, y esto lo lograremos gracias a dos comando:

Add-Migration MigrationNumber1 (el MigrationNumber1 puede cambiar por cualquier nombre la condicion es que no se debe repetir)

Update-Database (nos actualizara la base de datos)

 Despues de crear nuestro proyecto, elegir MVC. crearlo con la autenticacion de usuarios e instalar en Entity Framework tal y como lo vimos en el video, crearemos los modelos y en este caso seran 3.
 
 +Producto
 
    [Table("Producto")]

    public class Producto
    {
        public int id { get; set; }

        public string nombre { get; set; }

        public string descripcion { get; set; }

        public double precio { get; set; }
    }



 +Detalle 
 public class Detalle
    {
        public int detalleid { get; set; }

        public Cotizacion cotizacion { get; set; }
        public virtual Producto producto { get; set; }

        [ForeignKey("Producto")]
        public int productoid { get; set; }

        
        public int cantidad { get; set; }

        public double subtotal { get; set; }
    }
 
 +Cotizacion
 
 [Table("Cotizacion")]
    public class Cotizacion
    {
        
        
        public int cotizacionid { get; set; }
        
        
        public DateTime fecha { get; set; }

        
        public double total { get; set; }


    }
 
Y crearemos un extra con el nombre CotizacionViewModel que nos servira la cotizacion con la lista de detalle de productos y poder gurdar en la base la cotizacion con su detalle.
 
public class CotizacionViewModel
    {

        public DateTime fecha { get; set; }

        public double total { get; set; }
        public int productoid { get; set; }
        public int cantidad { get; set; }
        public List<Detalle> detalle { get; set; }
        public CotizacionViewModel() {
            detalle = new List<Detalle>();
        }

    }
    
    
Despues pasaremos a la creacion del de los controladores, pero para producto, Microsoft no da la posibilidad de generar un
CRUD automatico como se logra ver en el video. NO OLVIDEN PRIMERO COMPILAR EL  PROYECTO PARA QUE LA CREACION DEL CONTROLLER CON SUS VISTAS SEA EL CORRECTO.
  
  
  +Producto Controller
  
  [Authorize]
    public class ProductosController : Controller
    {
        private readonly ApplicationDbContext _context;

        public ProductosController(ApplicationDbContext context)
        {
            _context = context;
        }

        // GET: Productos
        public async Task<IActionResult> Index()
        {
            return View(await _context.Producto.ToListAsync());
        }

        // GET: Productos/Details/5
        public async Task<IActionResult> Details(int? id)
        {
            if (id == null)
            {
                return NotFound();
            }

            var producto = await _context.Producto
                .FirstOrDefaultAsync(m => m.id == id);
            if (producto == null)
            {
                return NotFound();
            }

            return View(producto);
        }

        // GET: Productos/Create
        public IActionResult Create()
        {
            return View();
        }

        // POST: Productos/Create
        // To protect from overposting attacks, please enable the specific properties you want to bind to, for 
        // more details see http://go.microsoft.com/fwlink/?LinkId=317598.
        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> Create([Bind("id,nombre,descripcion,precio")] Producto producto)
        {
            if (ModelState.IsValid)
            {
                _context.Add(producto);
                await _context.SaveChangesAsync();
                return RedirectToAction(nameof(Index));
            }
            return View(producto);
        }

        // GET: Productos/Edit/5
        public async Task<IActionResult> Edit(int? id)
        {
            if (id == null)
            {
                return NotFound();
            }

            var producto = await _context.Producto.FindAsync(id);
            if (producto == null)
            {
                return NotFound();
            }
            return View(producto);
        }

        // POST: Productos/Edit/5
        // To protect from overposting attacks, please enable the specific properties you want to bind to, for 
        // more details see http://go.microsoft.com/fwlink/?LinkId=317598.
        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> Edit(int id, [Bind("id,nombre,descripcion,precio")] Producto producto)
        {
            if (id != producto.id)
            {
                return NotFound();
            }

            if (ModelState.IsValid)
            {
                try
                {
                    _context.Update(producto);
                    await _context.SaveChangesAsync();
                }
                catch (DbUpdateConcurrencyException)
                {
                    if (!ProductoExists(producto.id))
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
            return View(producto);
        }

        // GET: Productos/Delete/5
        public async Task<IActionResult> Delete(int? id)
        {
            if (id == null)
            {
                return NotFound();
            }

            var producto = await _context.Producto
                .FirstOrDefaultAsync(m => m.id == id);
            if (producto == null)
            {
                return NotFound();
            }

            return View(producto);
        }

        // POST: Productos/Delete/5
        [HttpPost, ActionName("Delete")]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> DeleteConfirmed(int id)
        {
            var producto = await _context.Producto.FindAsync(id);
            _context.Producto.Remove(producto);
            await _context.SaveChangesAsync();
            return RedirectToAction(nameof(Index));
        }

        private bool ProductoExists(int id)
        {
            return _context.Producto.Any(e => e.id == id);
        }
    }
    
  y ahora crearemos el Cotizacion Controller, para hacer uso de loggin que ocupamos en el inicion en las funciones que queremos que solo puedan acceder las personas registradas tenemos que agregar un [Authorize] en las diferentes funciones creadas en el controlador que queremos controlar:
  
   public class CotizacionsController : Controller
    {
        private readonly ApplicationDbContext _context;
        public static List<Detalle> detalles = new List<Detalle>();
        public static double total = 0;
        public CotizacionsController(ApplicationDbContext context)
        {
            _context = context;

        }


        // GET: Cotizacions
        [Authorize]
        public async Task<IActionResult> Index()
        {
            return View(await _context.Cotizacion.ToListAsync());
        }



        // GET: Cotizacions/Create
        public IActionResult Create()
        {
            var viewModel = new CotizacionViewModel();
            // viewModel.detalle.Add(new Detalle() { cantidad=10, subtotal=1000});
            ViewBag.productoid = new SelectList(_context.Producto, "id", "nombre");
            ViewBag.total = total;
            viewModel.cantidad = 1;
            viewModel.fecha = DateTime.Now;
            detalles = new List<Detalle>();
            return View(viewModel);
        }


        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> Create([Bind("productoid,cantidad,fecha,total")] CotizacionViewModel cotizacion, string submit)
        {

            if (ModelState.IsValid)
            {
                if (!string.IsNullOrEmpty(submit))
                {

                    //agregar un producto a la lista del detalle de productos 
                    if (submit.Equals("agregar"))
                    {
                        //buscar producto
                        Producto producto = _context.Producto.Find(cotizacion.productoid);
                        //agregarlo a la lista
                        Detalle detalle = new Detalle();
                        detalle.producto = producto;// solo para visualizar
                        detalle.productoid = producto.id;
                        detalle.cantidad = cotizacion.cantidad;
                        detalle.subtotal = detalle.cantidad * producto.precio;
                        detalles.Add(detalle);

                        //actualizar valor total 
                        foreach (Detalle item in detalles)
                        {
                            total += item.subtotal;
                        }

                        cotizacion.detalle = detalles;//.Add(new Detalle() { cantidad = 10, subtotal = 1000 });
                        //cargar selects
                        ViewBag.total = total;
                        ViewBag.productoid = new SelectList(_context.Producto, "id", "nombre");
                        return View(cotizacion);
                    }
                    //guardar la cotizacion
                    if (submit.Equals("guardar"))
                    {
                        if (detalles != null && detalles.Count > 0)
                        {
                            //crear y guardar la cotizacion
                            Cotizacion cotizacion1 = new Cotizacion();
                            cotizacion1.fecha = cotizacion.fecha;
                            cotizacion1.total = total;

                            _context.Cotizacion.Add(cotizacion1);
                            _context.SaveChanges();
                            //crear y guardar los detalles
                            foreach (Detalle detalle in detalles)
                            {
                                detalle.cotizacion = cotizacion1;
                                Producto producto = new Producto();
                                producto.id = detalle.productoid;
                                detalle.producto = null;
                                detalle.productoid = producto.id;
                                _context.Detalle.Add(detalle);
                            }

                            await _context.SaveChangesAsync();
                            //redireccionar a la lista de cotizaciones
                            TempData["msg"] = "<script>alert('Cotizacion creada exitosamente');</script>";

                            return RedirectToAction(nameof(Index));
                        }
                        else
                        {
                            TempData["msg"] = "<script>alert('Agregue un producto');</script>";
                            ViewBag.total = total;
                            ViewBag.productoid = new SelectList(_context.Producto, "id", "nombre");
                            return View(cotizacion);

                        }

                    }
                }

            }
            return View(cotizacion);
        }

    }

Ahora pos ultimo viene la creacion de las vistas, en lo cual el Ya tenemos las 5 vistas de Producto ya que se generaron automaticamente con en el video.

Entonces tendremos que tener dos vistas para Cotizacion 

Unas que sera el Create que nos permitira ver la Cotizacion creada y otra que sera Index para poder ver las cotizaciones creadas.

+Create
@model WebApplicationProductos.Models.CotizacionViewModel

@{
    ViewData["Title"] = "Create";
}

    <h2>Cotizador</h2>

<hr />
  @Html.Raw(TempData["msg"])

<div class="row">
    <div class="col-md-4">
        <form method="post">
            <div asp-validation-summary="ModelOnly" class="text-danger"></div>
            <div class="form-group">
                <label asp-for="fecha" class="control-label"></label>

                <input asp-for="fecha"    class="form-control" type="date" />
                <span asp-validation-for="fecha"  class="text-danger"></span>
            </div>

            <div class="form-group">
                <label asp-for="total" class="control-label"></label>
                <input asp-for="total" value="@ViewBag.total" class="form-control" disabled="true" />

                <span asp-validation-for="total" class="text-danger"></span>
                <input type="submit" name="submit" value="guardar" class="btn btn-success" />

            </div>
            <hr />
            <div class="form-group">
                <h4>Agregar Producto</h4>
                @Html.LabelFor(model => model.productoid, "Producto", htmlAttributes: new { @class = "control-label col-md-2" })
                <div class="form-group">
                    @Html.DropDownList("productoid", null, htmlAttributes: new { @class = "form-control" })
                    @Html.ValidationMessageFor(model => model.productoid, "", new { @class = "text-danger" })
                </div>
                <div class="form-group">
                    <label asp-for="cantidad" class="control-label"></label>
                    <input asp-for="cantidad" class="form-control" min="1" type="number" />

                    <span asp-validation-for="total" class="text-danger"></span>
                </div>
                <input type="submit" name="submit" value="agregar" class="btn btn-info" />

            </div>

            <hr />
            Detalle
            <table class="table">
                <thead>
                    <tr>
                        <th>
                            producto
                        </th>
                        <th>
                            cantidad
                        </th>
                        <th>
                            total
                        </th>
                        <th></th>
                    </tr>
                </thead>
                <tbody>
                    @foreach (var item in Model.detalle)
                    {
                        <tr>
                            <td>
                                @Html.DisplayFor(modelItem => item.producto.nombre)
                            </td>
                            <td>
                                @Html.DisplayFor(modelItem => item.cantidad)
                            </td>
                            <td>
                                $@Html.DisplayFor(modelItem => item.subtotal)
                            </td>
                        </tr>
                    }
                </tbody>
            </table>

        </form>
    </div>
</div>

<div>
    <a asp-action="Index">Back to List</a>
</div>

@section Scripts {
    @{await Html.RenderPartialAsync("_ValidationScriptsPartial");}
}

+Index

@model IEnumerable<WebApplicationProductos.Models.Cotizacion>

@{
    ViewData["Title"] = "Index";
}

<h2>Listado de Cotizaciones</h2>


<table class="table">
    <thead>
        <tr>
            <th>
                @Html.DisplayNameFor(model => model.cotizacionid)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.fecha)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.total)
            </th>
            <th></th>
        </tr>
    </thead>
    <tbody>
@foreach (var item in Model) {
        <tr>
            <td>
                @Html.DisplayFor(modelItem => item.cotizacionid)
            </td>
            <td>
                @Html.DisplayFor(modelItem => item.fecha)
            </td>
            <td>
               $ @Html.DisplayFor(modelItem => item.total)
            </td>

        </tr>
}
    </tbody>
</table>

Por ultimo tenemos el Layaout que e sla pagina principal y la que nos dara el acceso a las diferentes funciones creadas por nosotros..Siempre debemos poner los nombres correctos de los Controllers si la palabra Controller al final y tambien apuntando a la vista, como podemos ver en el navbar-collapse collapse.

<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - VideoIngWebYoutube</title>

    <environment include="Development">
        <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.css" />
        <link rel="stylesheet" href="~/css/site.css" />
    </environment>
    <environment exclude="Development">
        <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/3.4.1/css/bootstrap.min.css"
              asp-fallback-href="~/lib/bootstrap/dist/css/bootstrap.min.css"
              asp-fallback-test-class="sr-only" asp-fallback-test-property="position" asp-fallback-test-value="absolute" />
        <link rel="stylesheet" href="~/css/site.min.css" asp-append-version="true" />
    </environment>
</head>
<body>
    <nav class="navbar navbar-inverse navbar-fixed-top">
        <div class="container">
            <div class="navbar-header">
                <button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-collapse">
                    <span class="sr-only">Toggle navigation</span>
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                </button>
                <a asp-area="" asp-controller="Home" asp-action="Index" class="navbar-brand">VideoIngWebYoutube</a>
            </div>
            <div class="navbar-collapse collapse">
                <ul class="nav navbar-nav">
                    <li><a asp-area="" asp-controller="Productos" asp-action="Index">Productos</a></li>
                    <li><a asp-area="" asp-controller="Cotizacions" asp-action="Create">Cotizador</a></li>
                    <li><a asp-area="" asp-controller="Cotizacions" asp-action="Index">Listado Cotizaciones</a></li>

                </ul>
                <partial name="_LoginPartial" />
            </div>
        </div>
    </nav>

    <partial name="_CookieConsentPartial" />

    <div class="container body-content">
        @RenderBody()
        <hr />
        <footer>
            <p>&copy; 2019 - VideoIngWebYoutube</p>
        </footer>
    </div>

    <environment include="Development">
        <script src="~/lib/jquery/dist/jquery.js"></script>
        <script src="~/lib/bootstrap/dist/js/bootstrap.js"></script>
        <script src="~/js/site.js" asp-append-version="true"></script>
    </environment>
    <environment exclude="Development">
        <script src="https://ajax.aspnetcdn.com/ajax/jquery/jquery-3.3.1.min.js"
                asp-fallback-src="~/lib/jquery/dist/jquery.min.js"
                asp-fallback-test="window.jQuery"
                crossorigin="anonymous"
                integrity="sha384-tsQFqpEReu7ZLhBV2VZlAu7zcOV+rXbYlF2cqB8txI/8aZajjp4Bqd+V6D5IgvKT">
        </script>
        <script src="https://stackpath.bootstrapcdn.com/bootstrap/3.4.1/js/bootstrap.min.js"
                asp-fallback-src="~/lib/bootstrap/dist/js/bootstrap.min.js"
                asp-fallback-test="window.jQuery && window.jQuery.fn && window.jQuery.fn.modal"
                crossorigin="anonymous"
                integrity="sha384-aJ21OjlMXNL5UyIl/XNwTMqvzeRMZH2w8c5cRVpzpU8Y5bApTppSuUkhZXN0VxHd">
        </script>
        <script src="~/js/site.min.js" asp-append-version="true"></script>
    </environment>

    @RenderSection("Scripts", required: false)
</body>
</html>
