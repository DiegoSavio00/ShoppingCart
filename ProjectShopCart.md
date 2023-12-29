> Projeto Shopping Cart!!!
> Tutorial AspNet.MVC 6

- Primeiramente vamos instalar as dependências do projeto(os packages).
- AspNetCore.Identity.EntityFramework
- AspNetCore.Mvc.NewtonsofJson  - Opcional
- EntityFrameworkCore.Design
- EntityFrameworkCore.SqlServer
- VisualStudio.Web.CodeGeneration.Design

- Agora vamos para o arquivo appsetting.json e vamos inserir a ConnectionString.
# Inserir abaixo do AllowdHost
        "ConnectionStrings": {
                "DbConnection": "Server=(localdb)\\MSSQLLocalDB;Database=ShoppingCart;MultipleActiveResultSets=True"
        }

- E vamos para o arquivo lauchSettings.json na pasta Properties.
- E alterar a porta para 3000 - "appplicationUrl" e "lauchBrowser" false

- Agora seria GitHub, mas não vamos usar.

- Então vamos começar criando nossa Infrastructure(infraestrutura)
- Que é a classe DataContext para utilizar o EF Core.
- Vamos criar uma pasta Infra e dentro dessa pastar criar a classe DataContext
# DataContext com DbContext e o construtor padrão.
    public class DataContext : DbContext
    {
        public DataContext(DbContextOptions<DataContext> options) : base(options)
        {
        }
    }

- E então vamos configurar na classe de configuração(Program).
# Abaixo do WebApplication.
    builder.Services.AddDbContext<DataContext>(options => 
    {
        options.UseSqlServer(builder.Configuration["ConnectionStrings:DbConnection"]);
    });

- Bora partir para os Models
- Dentro de Models vamos criar uma classe Product
# Classe Product e seus atributos.
    public class Product
    {
        public long Id { get; set; }
        public string Name { get; set; }
        public string Slug { get; set; }
        public string Description { get; set; }
        public decimal Price { get; set; }
        public long CategoryId { get; set; }             
        public Category Category { get; set; }           !> Faz o mapeamento com Product, ForeignKey <!
        public string Image { get; set; }
    }

- Vamos clicar no nome do projeto e no arquivo padrão de dependências, vamos alterar o <Nullable enable para disable.

- Ainda em Models vamos criar a classe Cateogry.
# Classe Category
    public class Category
    {
        public long Id { get; set; }
        public string Name { get; set; }
        public string Slug { get; set; }
    }

- Agora vamos voltar para classe Product e inseir algumas notações na classe.
# Anotações em Name, Description e Price.
    public class Product
    {
        public long Id { get; set; }
        [Required(ErrorMessage = "Please enter a value")]                !> 1 <!
        public string Name { get; set; }
        public string Slug { get; set; }
        [Required, MinLength(4, ErrorMessage = "Minimum lenght is 2")]    !> 2 <!
        public string Description { get; set; }
        [Required]                                                                  !> 3 <! 
        [Range(0.01, double.MaxValue, ErrorMessage = "Please enter a value")]       !> 3 <!
        [Column(TypeName = "decimal(8,2)")]                                         !> 3 <!
        public decimal Price { get; set; }
        public long CategoryId { get; set; }
        public Category Category { get; set; }
        public string Image { get; set; }
    }

- E então vamos para a DataContext configurar Product e Category em nosso DbContext.
- Vamos inserir os DbSet.
# Com as duas classes.
    public class DataContext : DbContext
    {
        public DbSet<Product> Products { get; set; }
        public DbSet<Category> Categories { get; set; }

# Opcional. Era pra da certo, mas não dá!!!
# Não da certo, não sei porque. kkkk
- Vamos clicar na aplicação com botão direito e Open in Terminal. Inserir...
- dotnet ef migrations add Initial

- Agora vamos fazer nosso primeiro Migrations.
# Então vamos para o PackageManager Console.
PM> Add-Migration Initial

- Vai gerar um arquivo c# com configurações Sql.
- Então vamos de mais um comando de Update
PM> Update-Database

- Vamos abrir o SqlServer.
- Vamos em View - SqlServer Object Explorer e verificar. 
- Até aqui tudo Ok!!

- Vamos criar uma classe SeedData
- Vamos copiar todo conteúdo da classe no repositorio do projeto original.
- Pois essa é uma classe para popular nosso DbContext, ou seja, já popular o Banco de dados com informações.
- Vamos a pasta Infra e criar a classe SeedData.
# Abaixo toda a classe SeedData com as categorias e os produtos.
        public static void SeedDatabase(DataContext context)
        {
            context.Database.Migrate();
            if (!context.Products.Any())
            {
                Category fruits = new Category { Name = "Fruits", Slug = "fruits" };
                Category shirts = new Category { Name = "Shirts", Slug = "shirts" };
                context.Products.AddRange(
                        new Product
                        {
                            Name = "Apples",
                            Slug = "apples",
                            Description = "Juicy apples",
                            Price = 1.50M,
                            Category = fruits,
                            Image = "apples.jpg"
                        },
                        new Product
                        {
                            Name = "Bananas",
                            Slug = "bananas",
                            Description = "Fresh bananas",
                            Price = 3M,
                            Category = fruits,
                            Image = "bananas.jpg"
                        },
                        new Product
                        {
                            Name = "Watermelon",
                            Slug = "watermelon",
                            Description = "Juicy watermelon",
                            Price = 0.50M,
                            Category = fruits,
                            Image = "watermelon.jpg"
                        },
                        new Product
                        {
                            Name = "Grapefruit",
                            Slug = "grapefruit",
                            Description = "Juicy grapefruit",
                            Price = 2M,
                            Category = fruits,
                            Image = "grapefruit.jpg"
                        },
                        new Product
                        {
                            Name = "White shirt",
                            Slug = "white-shirt",
                            Description = "White shirt",
                            Price = 5.99M,
                            Category = shirts,
                            Image = "white shirt.jpg"
                        },
                        new Product
                        {
                            Name = "Black shirt",
                            Slug = "black-shirt",
                            Description = "Black shirt",
                            Price = 7.99M,
                            Category = shirts,
                            Image = "black shirt.jpg"
                        },
                        new Product
                        {
                            Name = "Yellow shirt",
                            Slug = "yellow-shirt",
                            Description = "Yellow shirt",
                            Price = 11.99M,
                            Category = shirts,
                            Image = "yellow shirt.jpg"
                        },
                        new Product
                        {
                            Name = "Grey shirt",
                            Slug = "grey-shirt",
                            Description = "Grey shirt",
                            Price = 12.99M,
                            Category = shirts,
                            Image = "grey shirt.jpg"
                        }
                );
                context.SaveChanges();
            }
        }
    }

- Após criar os dados para nosso banco de dados.
- Vamos fazer a configuração da classe SeedData na classe Program.
# Na classe Program abaixo do app.MapController
    var context = app.Services.CreateScope().ServiceProvider.GetRequiredService<DataContext>();
    SeedData.SeedDatabase(context);

- Rodar a aplicação!! E ver o resultado!!
- http://localhost:3000
- Tudo Ok!!

- E agora vamos para os NotificationPartials
- Vamos para pastar Views e dentro da Pasta Shared criar um arquivo _NotificationPartial.cshtml
- E criar as mensagens de notificação de Sucesso e Error.
# Sucesso e Error no arquivo _NotificationPartial.cshtml
    @if (TempData["Success"] != null)
    {
        <div class="alert alert-success notification">
            @TempData["Sucess"]
        </div>
    }

    @if (TempData["Error"] != null)
    {
        <div class="alert alert-danger notification">
            @TempData["Error"]
        </div>
    }

- Continuando na mesma pasta, mas no arquivo _Layout
- Vamos localizar a <div que contém <main role="main" e inserir outra <div dentro desse main=role.
# Abaixo do <.main role="main" class="pb-3">
            <div class="row">
                <div class="col">
                    <partial name="_NotificationPartial" />
                    @RenderBody()
                </div>
            </div>

- Rodar a aplicação!! 

- E agora vamos criar o View Component.
- Vamos para a pastar Infra, criar uma nova pasta Components
- E criar uma classe CategoriesViewComponent que extende a ViewComponent
- Vamos fazer a injeção de dependência do DataContext.
- E vamos criar um método async com retorno do IViewComponentResult. IViewComponentResult retorna um componente view
# Classe CategoriesViewComponent e seu método de lista de categories.
    public class CategoriesViewComponent : ViewComponent
    {
        private readonly DataContext _context;
        public CategoriesViewComponent(DataContext context)
        {
            _context = context;
        }

        public async Task<IViewComponentResult> InvokeAsync() => View(await _context.Categories.ToListAsync());
    }

- Então vamos para a pasta Shared e criar uma pasta chamada Components
- E dentro da pasta Components vamos criar uma pasta Categories.
- E dentro da pasta Categories criar um arquivo Default.cshtml
# Arquivo Default. Todos os produtos. Ou seja, lista de todos os produtos e iterando com foreach.
    @model IEnumerable<Category>

    <ul class="list-group">
        <li class="list-group-item">
            <a class="nav-link text-dark" asp-controller="Products" asp-action="Index">All Products</a>
        </li>

        @foreach(var item in Model)
        {
            <li class="list-group-item">
                <a class="nav-link text-dark" asp-controller="Products" asp-action="Index">@item.Name</a>
            </li>
        }
    </ul>

- Vamos então para o arquivo _Layout.cshtml.
- Dentro da <div class ="row" que inserimos posteriomente. Criar outra div col.
# Abaixo do <.div class="row">
                <div class="col-3">
                    <vc:categories/>
                </div>

- Vamos então para o arquivo _ViewImports e importa nosso componente.
# Adicionar abaixo do outro addTagHelper
    @addTagHelper *, ShoppingCart

- Rodar a aplicação!!! E visualizar a aplicação.

- Contiamos a aplicação com os Controllers.
- Vamos criar ProductController.
- Vamos para pasta Controller e criar um controller ProductsController
- Vamos injetar o DataContext para nosso ProductsController
# O ProductsController
    public class ProductsController : Controller
    {
        private readonly DataContext _context;

        public ProductsController(DataContext context)
        {
            _context = context;
        }

        public IActionResult Index()
        {
            return View();
        }
    }

- E agora vamos criar uma classe para a ajudar na páginação.
- Dentro da pasta Infra vamos criar uma pasta TagHelpers
- E criar a classe PaginationTagHelper que vai extender a TagHelper.
# Toda a classe de página.
        public override void Process(TagHelperContext context, TagHelperOutput output)
        {
            output.TagName = "nav";
            output.TagMode = TagMode.StartTagAndEndTag;
            output.Attributes.Add("aria-label", "Page navigation");
            output.Content.SetHtmlContent(AddPageContent());
        }

        public int PageNumber { get; set; }
        public int PageSize { get; set; }
        public int PageCount { get; set; }
        public int PageRange { get; set; }
        public string PageFirst { get; set; }
        public string PageLast { get; set; }
        public string PageTarget { get; set; }

        private string AddPageContent()
        {
            if (PageRange == 0)
            {
                PageRange = 1;
            }
            if (PageCount < PageRange)
            {
                PageRange = PageCount;
            }
            if (string.IsNullOrEmpty(PageFirst))
            {
                PageFirst = "First";
            }
            if (string.IsNullOrEmpty(PageLast))
            {
                PageLast = "Last";
            }
            var content = new StringBuilder();
            content.Append(" <ul class='pagination'>");
            if (PageNumber != 1)
            {
                content.Append($"<li class='page-item'><a class='page-link' href='{PageTarget}'>{PageFirst}</a></li>");
            }
            if (PageNumber <= PageRange)
            {
                for (int currentPage = 1; currentPage < 2 * PageRange + 1; currentPage++)
                {
                    if (currentPage < 1 || currentPage > PageCount)
                    {
                        continue;
                    }
                    var active = currentPage == PageNumber ? "active" : "";
                    content.Append($"<li class='page-item {active}'><a class='page-link'href='{PageTarget}?p={currentPage}'>{currentPage}</a></li>");
                }
            }
            else if (PageNumber > PageRange && PageNumber < PageCount - PageRange)
            {
                for (int currentPage = PageNumber - PageRange; currentPage < PageNumber + PageRange; currentPage++)
                {
                    if (currentPage < 1 || currentPage > PageCount)
                    {
                        continue;
                    }
                    var active = currentPage == PageNumber ? "active" : "";
                    content.Append($"<li class='page-item {active}'><a class='page-link'href='{PageTarget}?p={currentPage}'>{currentPage}</a></li>");
                }
            }
            else
            {
                for (int currentPage = PageCount - (2 * PageRange); currentPage < PageCount + 1; currentPage++)
                {
                    if (currentPage < 1 || currentPage > PageCount)
                    {
                        continue;
                    }
                    var active = currentPage == PageNumber ? "active" : "";
                    content.Append($"<li class='page-item {active}'><a class='page-link'href='{PageTarget}?p={currentPage}'>{currentPage}</a></li>");
                }
            }
            if (PageNumber != PageCount)
            {
                content.Append($"<li class='page-item'><a class='page-link' href='{PageTarget}?p={PageCount}'>{PageLast}</a></li>");
            }
            content.Append(" </ul");
            return content.ToString();
        }
    }

- Então vamos voltar para nosso ProductsController e trabalhar no método Index
# Deixando de forma assincrono e inserindo as paginas e suas propriedades.
        public async Task<IActionResult> Index(string categorySlug = "", int p =1)
        {
            int pageSize = 3;
            ViewBag.PageNumber = p;
            ViewBag.PageRange = pageSize;
            ViewBag.CategorySlug = categorySlug;
            if (categorySlug == "")
            {
                ViewBag.TotalPages = (int)Math.Ceiling((decimal)_context.Products.Count() / pageSize);
                return View(await _context.Products.OrderByDescending(p => p.Id).Skip((p - 1) * pageSize).Take(pageSize).ToListAsync());
            }
            Category category = await _context.Categories.Where(c => c.Slug == categorySlug).FirstOrDefaultAsync();
            if (category == null) return RedirectToAction("Index");
            var productsByCategory = _context.Products.Where(p => p.CategoryId == category.Id);
            ViewBag.TotalPages = (int)Math.Ceiling((decimal)productsByCategory.Count() / pageSize);
            return View(await productsByCategory.OrderByDescending(p => p.Id).Skip((p - 1) * pageSize).Take(pageSize).ToListAsync());
        }

- Vamos agora para Products Index View
- Na pasta View vamos criar outra pasta Products e inserir arquivo Index 
- Podemos copiar Index da pasta Home.
- Então vamos trabalhar nesse IndexView da pasta Products
- Apagar tudo que está no Index e vamos códar.
# Todo código do arquivo Index.cshtml da pasta Products. Component de Products.
    @model IEnumerable<Product>

    @{
        ViewData["Title"] = "Products";
    }

    <h1>Products</h1>

    <div class="row">
        @foreach(var item in Model)
        {
            <div class="col-4">
                <img src="/media/products/@item.Image" class="img-fluid" alt=""/>
                <h4>@item.Name</h4>
                <div>
                    @Html.Raw(item.Description)
                </div>
                <p>
                    @item.Price.ToString("C2")
                </p>
                <p>
                    <a class="btn btn-primary" href="#">Add to cart</a>
                </p>
            </div>
        }

        @if(ViewBag.TotalPages > 1)
        {
            <div class="d-flex w-100 justify-content-center">
                <pagination> 
                    page-count="@ViewBag.TotalPages" 
                    page-target="/products/@ViewBag.CategorySlug" 
                    page-number="@ViewBag.PageNumber"
                    page-range="@ViewBag.PageRange">
                </pagination>
            </div>
        }
    </div>

- Após o component de Products. Vamos para Pasta Shared e depois para pasta Categories.
- Vamos trabalhar no arquivo Default dentro da pasta Categories.
- Vamos inserir a rota do nosso categorySlug.
# Somente a linha a -> asp-route-categorySlug="@item.Slug"
    <a class="nav-link text-dark" asp-controller="Products" asp-action="Index" asp-route-categorySlug="@item.Slug">@item.Name</a>

- Vamos rodar a aplicação!!!

- Vamos então configurar outro MapController, no caso o de Products.
# Inserir acima do outro MapController
    app.MapControllerRoute(
        name: "products",
        pattern: "/products/{categorySlug?}",
        defaults: new { controller = "Products", action = "Index" });

- Rodar a aplicação!!!
- Temos o probleminha com rota no All Products e então vamos inserir também asp-route-categorySlug
- Vamos no arquivo Default da pasta Categories em Shared.
# Na primeira li de All Products -> asp-route-categorySlug=""
    <a class="nav-link text-dark" asp-controller="Products" asp-action="Index" asp-route-categorySlug="">All Products</a>

# Até Aqui tudo certo!!

- Vamos para o wwwroot e criar uma pasta com nome media e dentro de media criar uma pasta products
- Dentro da pasta products vamos inserir todas as imagens dos fruits. 
- Kiwi pode apagar.

- Agora SessionExtensions
- Vamos pasta Infra e criar a classe SessionExtension 
- Essa classe será static, pois vai obter os métodos de extensão.  
# Médoto setar o objeto Json e obter o objeto Json.
    public static class SessionExtensions
    {
        public static void SetJson(this ISession session, string key, object value)
        {
            session.SetString(key, JsonConvert.SerializeObject(value));
        }

        public static T GetJson<T>(this ISession session, string key)
        {
            var sessionData = session.GetString(key);
            return sessionData == null ? default(T) : JsonConvert.DeserializeObject<T>(sessionData);
        }
    }

- E então vamos configurar esse serviço.
- Vamos para classe Program.
# Adicionar abaixo do -> builder.Services.AddDbContext<DataContext.>
    builder.Services.AddDistributedMemoryCache();
    builder.Services.AddSession(options =>
    {
        options.IdleTimeout = TimeSpan.FromMinutes(30);
        options.Cookie.IsEssential = true;
    });

- E também vamos configurar o UseSession();
# Abaixo do var app = builder.Build();
    app.UseSession();

- E agora vamos criar o SmallCartViewComponent
- Na pasta Models vamos criar outra pasta ViewModels.
- E dentro de ViewModels criar a classe SmallCartViewModel
# Classe SmallCartViewModel e atributos.
    public class SmallCartViewModel
    {
        public int NumberOfItems { get; set; }
        public decimal TotalAmount { get; set; }
    }

- E na pasta Models vamos criar uma classe CartItem
- E vamos inserir vários atributos.
# Classe CartItem.
    public class CartItem
    {
        public long ProductId { get; set; }
        public string ProductName { get; set; }
        public int Quantity { get; set; }
        public decimal Price { get; set; }
        public decimal Total { get { return Quantity * Price; } }
        public string Image { get; set; }

        public CartItem()
        {
        }

        public CartItem(Product product)
        {
            ProductId = product.Id;
            ProductName = product.Name;
            Price = product.Price;
            Quantity = 1;
            Image = product.Image;
        }
    }

- Apos criarmos as classes de CartItem e SmallCartViewModel
- Vamos para pasta Components na pasta Infra e criar o SmallCartViewComponent que extende a ViewComponent
# Classe SmallSmartViewComponent
    public class SmallCartViewComponent : ViewComponent
    {
        public IViewComponentResult Invoke()
        {
            List<CartItem> cart = HttpContext.Session.GetJson<List<CartItem>>("Cart");
            SmallCartViewModel smallCartVM;
            if (cart == null || cart.Count == 0)
            {
                smallCartVM = null;
            }
            else
            {
                smallCartVM = new()
                {
                    NumberOfItems = cart.Sum(x => x.Quantity),
                    TotalAmount = cart.Sum(x => x.Quantity * x.Price)
                };
            }
            return View(smallCartVM);
        }   
    }

- Após criado a classe SmallSmartViewComponent.
- Vamos para Views e dentro de Views localizar a pasta Shared e dentro de Shared temos Components
- Dentro da pasta Components de Views vamos criar outra pasta SmallCart
- E vamos criar um arquivo Default.cshmtl, igual fizemos com Categories.
- Mas claro vamos inserir alguns códigos.
- Ao trabalhar no arquivo temos que fazer o import no arquivo _ViewImports e inserir
# @using ShoppingCart.Models.ViewModels
# E então vamos trabalhar no arquivo Default de SmallCart
    @model SmallCartViewModel

    @if (Model != null)
    {
        <p>Items in cart: <span class="badge badge-primary">@Model.NumberOfItems</span></p>
        <p>Total: <b>@Model.TotalAmount.ToString("C2")</b></p>
        <a href="#" class="btn btn-primary mb-2">View Cart</a>
        <a href="#" class="btn btn-danger">Clear Cart</a>
    }
    else 
    {
        <p class="lead m-0">Your cart is empty</p>
    }

- E agora vamos inseir no Layout. Vamos para o arquivo _Layout
- Abaixo do <vc:categories/>
- Vamos inserir nosso component.
# Inserir a div abaixo do vc:categories
    <vc:categories/>
    <div class="bg-success mt-3 p-3 text-white">
        <vc:small-cart/>
    </div>

- Rodar a aplicação!!!
- Tudo Ok!!

- Agora vamos fazer o CartIndex e CartController.
- Vamos para Models e dentro da pasta ViewModels vamos criar outra classe CartViewModel
# Classe CartViewModel
    public class CartViewModel
    {
        public List<CartItem> CartItems { get; set; }
        public decimal GrandTotal { get; set; }
    }

- Então vamos para o Controllers e criar nosso CartController.
- Fazer a injeção de dependencia do DataContext e também será semelhante a ProductsController.
# CartController
    public class CartController : Controller
    {
        private readonly DataContext _context;
        public CartController(DataContext context)
        {
            _context = context;
        }

        public IActionResult Index()
        {
            List<CartItem> cart = HttpContext.Session.GetJson<List<CartItem>>("Cart") ?? new List<CartItem>();
            CartViewModel cartVM = new()
            {
                CartItems = cart,
                GrandTotal = cart.Sum(x => x.Quantity * x.Price)
            };
            return View(cartVM);
        }
    }

- Então agora vamos para Views e criar pasta Cart
- E dentro da pasta Cart criar também um arquivo component Index.cshtml.
- Esse Index será parecido com Index de Products.
#
    @model CartViewModel

    @{
        ViewData["Title"] = "Cart Overview";
    }

    @if (Model.CartItems.Count > 0)
    {
        <table class="table">
        <tr>
            <th>Product</th>
            <th>Quantity</th>
            <th></th>
            <th>Price</th>
            <th>Total</th>
        </tr>
        @foreach (var item in Model.CartItems)
        {
                <tr>
                    <td>@item.ProductName</td>
                    <td>@item.Quantity</td>
                    <td>
                        <a class="btn btn-primary btn-sm" asp-action="Add" asp-route-id="@item.ProductId">+</a>
                        <a class="btn btn-info btn-sm" asp-action="Decrease" asp-route-id="@item.ProductId">-</a>
                        <a class="btn btn-danger btn-sm" asp-action="Remove" asp-route-id="@item.ProductId">Remove</a>
                    </td>
                    <td>@item.Price.ToString("C2")</td>
                    <td>@Model.CartItems.Where(x => x.ProductId == item.ProductId).Sum(x => x.Quantity * x.Price).ToString("C2")</td>
                </tr>
        }
        <tr>
            <td class="text-right" colspan="4">Grand Total:@Model.GrandTotal.ToString("C2")</td>
        </tr>
        <tr>
            <td class="text-right" colspan="4">
                <a class="btn btn-danger" asp-action="Clear">Clear Cart</a>
                <a class="btn btn-primary" href="#">Checkout</a>
            </td>
        </tr>
        </table>
    }   
    else
    {
        <h3 class="display-4 text-center">Your cart is empty.</h3>
    }

- Rodar a aplicação!!! LocalHost:3000/cart
- Tudo Ok!!

- Então agora vamos para AddToCart, adicionar ao Cart.
- E agora vamos voltar para CartController e criar mais um método.
# Vamos criar um método de adicionar item ao Cart.
        public async Task<IActionResult> Add(long id)
        {
            Product product = await _context.Products.FindAsync(id);
            List<CartItem> cart = HttpContext.Session.GetJson<List<CartItem>>("Cart") ?? new List<CartItem>();
            CartItem cartItem = cart.Where(p => p.ProductId == id).FirstOrDefault();
            if (cartItem == null)
            {
                cart.Add(new CartItem(product));
            }
            else
            {
                cartItem.Quantity += 1;
            }
            HttpContext.Session.SetJson("Cart", cart);
            TempData["Sucess"] = "The product has been added";
            return Redirect(Request.Headers["Referer"].ToString());
        }

- Vamos para pasta Products dentro de Views e trabalhar no arquivo Index.cshmtl(no View).
- No ultimo <.p> do @foreach, vamos remover o href="#" e vamos inserir um asp-controller.
# Somente o ultimo parafrafo do foreach. apagando o href e inserindo asp-controller e action.
    <p>
        <a class="btn btn-primary" asp-controller="Cart" asp-action="Add" asp-route-id="@item.Id"->Add to cart</a>
    </p>

- Rodar a aplicação!! Add To Cart e verificar se está Ok!!
- Tudo Ok!!

- Vamos para o SmallCart. Shared - Components e SmallCart e temos o arquivo Default
- Que é o de adicionar item. Vamos inserir asp-controller no View Cart e remover o href="#".
# Somente a linha de <.a> de View Cart
    <a class="btn btn-primary" asp-controller="Cart" asp-action="Index">View Cart</a>

- Rodar aplicação!!! E assim mostra os que foi adicionado.
- Tudo Ok!!

- Então agora precisamos trabalhar em Decrease Remove e Clear Cart.
- Vamos voltar para o CartController e criar o método Descrease.
# 
        public async Task<IActionResult> Decrease(long id)
        {
            List<CartItem> cart = HttpContext.Session.GetJson<List<CartItem>>("Cart");
            CartItem cartItem = cart.Where(p => p.ProductId == id).FirstOrDefault();
            if (cartItem.Quantity > 1)
            {
                --cartItem.Quantity;
            }
            else
            {
                cart.RemoveAll(p => p.ProductId == id);
            }
            if (cart.Count == 0)
            {
                HttpContext.Session.Remove("Cart");
            }
            else
            {
                HttpContext.Session.SetJson("Cart", cart);
            }           
            TempData["Sucess"] = "The product has been removed";
            return RedirectToAction("Index");
        }

- Agora vamos criar o método de Remove do nosso CartController.
# Método Remove
        public async Task<IActionResult> Remove(long id)
        {
            List<CartItem> cart = HttpContext.Session.GetJson<List<CartItem>>("Cart");
            cart.RemoveAll(p => p.ProductId == id);
            if (cart.Count == 0)
            {
                HttpContext.Session.Remove("Cart");
            }
            else
            {
                HttpContext.Session.SetJson("Cart", cart);
            }
            TempData["Sucess"] = "The product has been removed";
            return RedirectToAction("Index");
        }

- Agora vamos criar o Clear.
# Método Clear
        public IActionResult Clear()
        {
            HttpContext.Session.Remove("Cart");
            return RedirectToAction("Index");
        }

- Temos um problema no botão de Clear Cart e então vamos resolver.
- Vamos em Views - Shared - Components - SmallCart e no arquivo Default vamos adicionar controller e action.
- Vamos fazer similar ao View Cart.
# Adicionando asp-controller e asp-action. 
    <a class="btn btn-danger" asp-controller="Cart" asp-action="Clear">Clear Cart</a>

- Rodar a aplicação!! E tudo Ok!!

# Testar a aplicação daqui pra frente, pois podemos ter um error futuro!!!

- Então vamos para o Admin ProductsController e Index
- Vamos criar um pasta Areas no nível do Projeto.
- E dentro da pasta Areas criar a pasta Admin. Na pasta Admin criar a pasta Controllers.
- Dentro da pasta Controllers criar o controller ProductsController.
# Controller Area
    [Area("Admin")]
    public class ProductsController : Controller
    {
        private readonly DataContext dataContext;

        public ProductsController(DataContext dataContext)
        {
            this.dataContext = dataContext;
        }

        public IActionResult Index()
        {
            return View();
        }
    }

- Vamos configurar o controller no Program.
# Inserir ACIMA do appMapControllerRoute de products
    app.MapControllerRoute(
        name: "Areas",
        pattern: "{area:exists}/{controller=Products}/{action=Index}/{id?}");

- Voltamos para o Controller de Area e vamos continuar trabalhando nele.
- Aqui precisamos somente dos Products, não precisa de Category.
# Somente o método Index.
        public async Task<IActionResult> Index(int p = 1)
        {
            int pageSize = 3;
            ViewBag.PageNumber = p;
            ViewBag.PageRange = pageSize;
            ViewBag.TotalPages = (int)Math.Ceiling((decimal)_context.Products.Count() / pageSize);
            return View(await _context.Products.OrderByDescending(p => p.Id)
                .Include(p => p.Category)
                .Skip((p - 1) * pageSize)
                .Take(pageSize)
                .ToListAsync());
        }

- Vamos clicar com botão direito do mouse escolher Add View.
- Selecionar Razor View e Inserir os dados.
- View Name = Index, Template = List, Model = Product, DataContext e Add.
- Vai criar uma pasta Views com Products e dentro dessa pasta tem o arquivo Index.cshmtl.
- Vamos copiar os arquivos _ViewImport e _ViewStar da pasta View padrão.
- E colar dentro da nossa nova pasta View da Areas.

- Vamos também copiar o Layout para inserir em Areas.
- Vamos copiar a pasta Shared de Views e inserir na parte de View de Areas
- Vamos deletar a pasta Component que copiamos de Shared

- Vamos para a pasta Products da parte de Area na pasta Views.
- E trabalhar no arquivo que criarmos. No caso é o Razor View.
# Alteração do Arquivo Razor View que gerando.
# E no fim do arquivo vamos inserir. Vamos para a pasta Products do View Padrão e copiar todo <div de ViewBag.TotalPage
# Inserir abaixo de <table
    @model IEnumerable<Product>

    @{
        ViewData["Title"] = "Product";
    }

    <h1>Products</h1>

    <p>
        <a asp-action="Create">Create New</a>
    </p>
    <table class="table">
        <thead>
            <tr>
                <th>Name</th>
                <th>Price</th>
                <th>Category</th>
                <th>Image</th>
                <th></th>
            </tr>
        </thead>
        <tbody>
    @foreach (var item in Model) {
            <tr>
                <td>
                    @item.Name
                </td>
                <td>
                    @item.Price.ToString("C2")
                </td>
                <td>
                    @item.Category.Name
                </td>
                <td>
                    <img src="/media/products/@item.Image" width="100" alt=""/>
                </td>
                <td>
                    <a asp-action="Edit" asp-route-id="@item.Id">Edit</a> |
                    <a asp-action="Delete" asp-route-id="@item.Id" class="confirmDeletion">Delete</a>
                </td>
            </tr>
    }
        </tbody>
    </table>

    <div clas="d-flex w-100 justify-content-center">
        <pagination 
            page-count="@ViewBag.TotalPages" 
            page-target="/admin/products/@ViewBag.CategorySlug" 
            page-number="@ViewBag.PageNumber"
            page-range="@ViewBag.PageRange">
        </pagination>
    </div>

- Então vamos para o arquivo Layout da parte de Areas.
- Layout está na pasta Views - Shared.
- Vamos localizar no arquivo Layout <title e temos ShoppingCart.
- Vamos apagar ShoppingCart e inserir Admin Area
# Alterado
    <title>@ViewData["Title"] - Admin Area</title>

- Continuando no arquivo, vamos apagar na div ="container", Toda a parte abaixo de <div ="col-3"
- Também apagar o footer, pois não faz sentido.
- Vamos remover também a parte de <li que é a lista. E inserir target="_blank no <a class="navbar-brand
# Qualquer duvida olhe o arquivo _Layout da parte de Area na pasta Shared.
# A pasta Views deve estar dentro da pasta Admin!!

- Rodar a aplicação!!! localhost:3000/admin
- Ate aqui tudo Ok!!

- Então vamos para o FileExtensionAttribute

- Vamos para pasta Infra e criar uma pasta Validation
- Dentro de Validation criar a classe FileExtensionAttribute herdando ValidationAttribute
- Sobrescrevendo o método ValidationResult.
# Classe FileExtensionAttribute
    public class FileExtensionAttribute : ValidationAttribute
    {
        protected override ValidationResult IsValid(object value, ValidationContext validationContext)
        {
            if (value is IFormFile file)
            {
                var extension = Path.GetExtension(file.FileName);
                string[] extensions = { "jpg", "png" };
                bool result = extensions.Any(x => extension.EndsWith(x));
                if (!result)
                {
                    return new ValidationResult("Allowed extensions are jpg and png");
                }
            }
            return ValidationResult.Success;
        }
    }

- Agora vamos inserir nova propriedade na classe Product na classe Models.
# Atributo da classe Product
        [NotMapped]
        [FileExtensions]
        public IFormFile ImageUpload { get; set; }

- Então vamos para o controlador ProductsController da area de Admin...Admin - Controllers
- Injetar IWebHostEnvironment e inserir dentro do construtor do controlador.
- E criar um novo método Create
# Controlador ProductsController da Area - Admin - Controllers
# Somente os código adicionado no controllador.
    private readonly IWebHostEnvironment _webHostEnvironmet;                                      -> Injeção de dependencia
    public ProductsController(DataContext context, IWebHostEnvironment webHostEnvironmet)         -> Construtor
    {
        _context = context;
        _webHostEnvironmet = webHostEnvironmet;
    }
    public IActionResult Create()                                                                 -> Método Create
    {
        ViewBag.Categories = new SelectList(_context.Categories, "Id", "Name");
        return View();
    }

- Vamos criar a view desse método.
- Pelo Scaffolding já criar a view ou criamos de forma manual. Na pasta Views - Products de Areas.
## Código completo da View Create
@model ShoppingCart.Models.Product
@{
        ViewData["Title"] = "Create";
}
<h1>Create Product</h1>
<div class="row">
        <div class="col-md-4">
                <form asp-action="Create" enctype="multipart/form-data">
                        <div asp-validation-summary="ModelOnly" class="text-danger"></div>
                        <div class="form-group">
                                <label asp-for="Name" class="control-label"></label>
                                <input asp-for="Name" class="form-control" />
                                <span asp-validation-for="Name" class="text-danger"></span>
                        </div>
                        <div class="form-group">
                                <label asp-for="Description" class="control-label"></label>
                                <textarea asp-for="Description" class="form-control"></textarea>
                                <span asp-validation-for="Description" class="text-danger"></span>
                        </div>
                        <div class="form-group">
                                <label asp-for="Price" class="control-label"></label>
                                <input asp-for="Price" class="form-control" />
                                <span asp-validation-for="Price" class="text-danger"></span>
                        </div>
                        <div class="form-group">
                                <label>Categories</label>
                                <select asp-for="CategoryId" class="form-control" asp-items="ViewBag.Categories">
                                        <option value="0">Choose a category</option>
                                </select>
                                <span asp-validation-for="CategoryId" class="text-danger"></span>
                        </div>
                        <div class="form-group">
                                <label asp-for="Image" class="control-label"></label>
                                <input asp-for="ImageUpload" class="form-control" />
                                <img id="imgpreview" class="pt-2" />
                                <span asp-validation-for="ImageUpload" class="text-danger"></span>
                        </div>
                        <div class="form-group">
                                <button class="btn btn-primary">Create</button>
                        </div>
                </form>
        </div>
</div>
<div>
        <a asp-action="Index">Back to List</a>
</div>
@section Scripts {
    @{
        await Html.RenderPartialAsync("_ValidationScriptsPartial");
}
<script src="https://cdn.ckeditor.com/4.19.0/standard/ckeditor.js"></script>
<script>
        $("#ImageUpload").change(function() {
                readURL(this);
        });
        CKEDITOR.replace( 'Description' );
</script>
}

# Código acima é da View Create de Products da Area
# Podemos já visualizar o arquivo completo!!

- E ir para a pasta wwwroot - na pasta Js e inserir o código abaixo!!
# Código site Js!!
        ﻿$(function () {
            if ($("a.confirmDeletion").length) {
                    $("a.confirmDeletion").click(() => {
                            if (!confirm("Confirm deletion")) return false;
                    });
            }
            if ($("div.alert.notification").length) {
                    setTimeout(() => {
                            $("div.alert.notification").fadeOut();
                    }, 2000);
            }
        });

    function readURL(input) {
        if (input.files && input.files[0]) {
                let reader = new FileReader();
                reader.onload = function (e) {
                        $("img#imgpreview").attr("src", e.target.result).width(200).height(200);
                };
                reader.readAsDataURL(input.files[0]);
        }
    }   

- Rodar a aplicação e testar!!! no http://localhost:????/admin no Create New.
- Tudo Ok até aqui!!!

- Continuando. Vamos voltar para o controlador ProductsController da Area - Admin - Controllers
- E sobrescrever o método Create, assim criando outro método Create.
# Método Create abaixo!!
                [HttpPost]
                [ValidateAntiForgeryToken]
                public async Task<IActionResult> Create(Product product)
                {
                        ViewBag.Categories = new SelectList(_context.Categories, "Id", "Name", product.CategoryId);
                        if (ModelState.IsValid)
                        {
                                product.Slug = product.Name.ToLower().Replace(" ", "-");
                                var slug = await _context.Products.FirstOrDefaultAsync(p => p.Slug == product.Slug);
                                if (slug != null)
                                {
                                        ModelState.AddModelError("", "The product already exists.");
                                        return View(product);
                                }
                                if (product.ImageUpload != null)
                                {
                                        string uploadsDir = Path.Combine(_webHostEnvironment.WebRootPath, "media/products");
                                        string imageName = Guid.NewGuid().ToString() + "_" + product.ImageUpload.FileName;
                                        string filePath = Path.Combine(uploadsDir, imageName);
                                        FileStream fs = new FileStream(filePath, FileMode.Create);
                                        await product.ImageUpload.CopyToAsync(fs);
                                        fs.Close();
                                        product.Image = imageName;
                                }
                                _context.Add(product);
                                await _context.SaveChangesAsync();
                                TempData["Success"] = "The product has been created!";
                                return RedirectToAction("Index");
                        }
                        return View(product);
                }

- Rodar a aplicação e testar!!! no http://localhost:????/admin/create
- Tudo Ok!!

- Vamos para a classe Products da pasta Models e no atributo CategoryId vamos inserir as data annotations.
# Atributo e annotation.
    [Required, Range(1, int.MaxValue, ErrorMessage = "You must choose a category")]
    public long CategoryId { get; set; }

- Rodar a aplicação!! Verificar se está Ok!!

- Agora vamos criar o método Edit no controlador ProductsController da Area de Admin em Controlllers.
# Método Edit Abaixo!!
                public async Task<IActionResult> Edit(long id)
                {
                        Product product = await _context.Products.FindAsync(id);
                        ViewBag.Categories = new SelectList(_context.Categories, "Id", "Name", product.CategoryId);
                        return View(product);
                }

- Vamos criar a view desse método.
- A View do método Edit.
- Pelo Scaffolding já criar a view ou criamos de forma manual. Na pasta Views - Products de Areas.
## View Edit Completa abaixo!! 
@model ShoppingCart.Models.Product
@{
    ViewData["Title"] = "Create";
}
<h1>Edit Product</h1>
<div class="row">
        <div class="col-md-4">
                <form asp-action="Edit" enctype="multipart/form-data">
                        <div asp-validation-summary="ModelOnly" class="text-danger"></div>
                        <input type="hidden" asp-for="Image" />
                        <div class="form-group">
                                <label asp-for="Name" class="control-label"></label>
                                <input asp-for="Name" class="form-control" />
                                <span asp-validation-for="Name" class="text-danger"></span>
                        </div>
                        <div class="form-group">
                                <label asp-for="Description" class="control-label"></label>
                                <textarea asp-for="Description" class="form-control"></textarea>
                                <span asp-validation-for="Description" class="text-danger"></span>
                        </div>
                        <div class="form-group">
                                <label asp-for="Price" class="control-label"></label>
                                <input asp-for="Price" class="form-control" />
                                <span asp-validation-for="Price" class="text-danger"></span>
                        </div>
                        <div class="form-group">
                                <label>Categories</label>
                                <select asp-for="CategoryId" class="form-control" asp-items="ViewBag.Categories">
                                        <option value="0">Choose a category</option>
                                </select>
                                <span asp-validation-for="CategoryId" class="text-danger"></span>
                        </div>
                        <div class="form-group">
                                <label>Current Image</label>
                                <img src="/media/products/@Model.Image" width="200" alt="" />
                        </div>
                        <div class="form-group">
                                <label asp-for="Image" class="control-label"></label>
                                <input asp-for="ImageUpload" class="form-control" />
                                <img id="imgpreview" class="pt-2" />
                                <span asp-validation-for="ImageUpload" class="text-danger"></span>
                        </div>
                        <div class="form-group">
                                <button class="btn btn-primary">Edit</button>
                        </div>
                </form>
        </div>
</div>

<div>
    <a asp-action="Index">Back to List</a>
</div>

@section Scripts {
    @{
        await Html.RenderPartialAsync("_ValidationScriptsPartial");
    }
    <script src="https://cdn.ckeditor.com/4.19.0/standard/ckeditor.js"></script>
    <script>
        $("#ImageUpload").change(function() {
            readURL(this);
        });
        CKEDITOR.replace( 'Description' );
    </script>
}

# Se a View List der error, visualize o arquivo principal no GitHub.

# View Edit Completa acima!!
# Rodar a aplicação e testar!!

- Continuando vamos voltar para o controlador ProductsController da Area - Admin - Controllers
- E criar sobrescrever o método Edit, assim criar outro método Edit.
# Esse método é o POST de Edit.
                [HttpPost]
                [ValidateAntiForgeryToken]
                public async Task<IActionResult> Edit(Product product)
                {
                        ViewBag.Categories = new SelectList(_context.Categories, "Id", "Name", product.CategoryId);
                        if (ModelState.IsValid)
                        {
                                product.Slug = product.Name.ToLower().Replace(" ", "-");
                                var slug = await _context.Products.FirstOrDefaultAsync(p => p.Slug == product.Slug);
                                if (slug != null)
                                {
                                        ModelState.AddModelError("", "The product already exists.");
                                        return View(product);
                                }
                                if (product.ImageUpload != null)
                                {
                                        string uploadsDir = Path.Combine(_webHostEnvironment.WebRootPath, "media/products");
                                        string imageName = Guid.NewGuid().ToString() + "_" + product.ImageUpload.FileName;
                                        string filePath = Path.Combine(uploadsDir, imageName);
                                        FileStream fs = new FileStream(filePath, FileMode.Create);
                                        await product.ImageUpload.CopyToAsync(fs);
                                        fs.Close();
                                        product.Image = imageName;
                                }
                                _context.Update(product);
                                await _context.SaveChangesAsync();
                                TempData["Success"] = "The product has been edited!";
                        }
                        return View(product);
                }

- Antes de testar vamos so fazer uma pequena alteração na propriedade Image da classe Product da pasta Models
# Caso não queira inserir uma imagem, vamos deixar a noimage.png como padrão.
    public string Image { get; set; } = "noimage.png";

- Rodar a aplicação!! Testar!!

- Continuando vamos voltar para o controlador ProductsController da Area - Admin - Controllers
- E criar sobrescrever o método Delete, assim criar outro método Delete.
# Método Delete!!
                public async Task<IActionResult> Delete(long id)
                {
                        Product product = await _context.Products.FindAsync(id);
                        if (!string.Equals(product.Image, "noimage.png"))
                        {
                                string uploadsDir = Path.Combine(_webHostEnvironment.WebRootPath, "media/products");
                                string oldImagePath = Path.Combine(uploadsDir, product.Image);
                                if (System.IO.File.Exists(oldImagePath))
                                {
                                        System.IO.File.Delete(oldImagePath);
                                }
                        }
                        _context.Products.Remove(product);
                        await _context.SaveChangesAsync();
                        TempData["Success"] = "The product has been deleted!";
                        return RedirectToAction("Index");
                }

- Rodar a aplicação!! Testar!!

# Atenção!! Não tem na pasta do projeto os códigos abaixo!!
# Verificar a branch do projeto no GitHub e acompanhar o vídeo, caso algo esteja errado!!

- Agora vamos iniciar a implementação do Identity na aplicação!!
- Vamos instalar os packages do Identity.
- Caso esteja instaldo vamos prosseguir.

- Na classe de contexto, ou seja, a classe DataContext vamos inserir o IdentityDbContext
# Somente alterando DbContext para IdentityDbContext
    public class DataContext : IdentityDbContext

- Vamos para a pasta Models e criar um novo modelo.
# Classe AppUser com IdentityUser
        public class AppUser : IdentityUser
        {
                public string Occupation { get; set; }
        }

- Vamos ter que voltar para a classe de contexto DataContext e inserir o AppUser
# Inserir como um tipo.
    public class DataContext : IdentityDbContext<AppUser>

- Vamos configurar o middleware na classe Program.
- Que é a configuração de senhas e registros do Identity.
# Vamos inserir abaixo do AddSession!!
    builder.Services.AddIdentity<AppUser, IdentityRole>().AddEntityFrameworkStores<DataContext>().AddDefaultTokenProviders();
    builder.Services.Configure<IdentityOptions>(options =>
    {
        options.Password.RequiredLength = 4;
        options.Password.RequireNonAlphanumeric = false;
        options.Password.RequireLowercase = false;
        options.Password.RequireUppercase = false;
        options.Password.RequireDigit = false;
        options.User.RequireUniqueEmail = true;
    });

- E acima do app.UseAuthorization(). Vamos inserir o app.UseAuthentication()
# Na classe Program acima do app.UseAuthorization()
    app.UseAuthentication();

- Vamos da um Drop no banco de dados.
- PM> Drop-Database 

- E vamos fazer uma nova migração.
- PM> Add-Migration Second

- PM> Update-Database

- Então agora vamos criar AccountController e também começar a trabalhar no registro de Admin e Login
- Vamos para a pasta Controllers e criar o controlador AccountController
# Todo o código abaixo.
# Primeiro vamos criar só o controllador e depois vamos inserir todo o código.
        public class AccountController : Controller
        {
        }

- Vamos para a pasta Models e criar o modelo User.
# Classe User e suas propriedades
        public class User
        {
                public string Id { get; set; }

                [Required, MinLength(2, ErrorMessage = "Minimum length is 2")]
                [Display(Name = "Username")]
                public string UserName { get; set; }
                [Required, EmailAddress]
                public string Email { get; set; }
                [DataType(DataType.Password), Required, MinLength(4, ErrorMessage = "Minimum length is 4")]
                public string Password { get; set; }
        }

- E na pasta ViewModels que está dentro da pasta Models.
- Criar a classe LoginViewModel
# Classe abaixo!!
        public class LoginViewModel
        {
                [Required, MinLength(2, ErrorMessage = "Minimum length is 2")]
                [Display(Name = "Username")]
                public string UserName { get; set; }

                [DataType(DataType.Password), Required, MinLength(4, ErrorMessage = "Minimum length is 4")]
                public string Password { get; set; }

                public string ReturnUrl { get; set; }
        }

- E então vamos voltar para o AccountController e trabalhar no controlador.
- Vamos fazer a injeção de dependencia do UserManager
- Criar o método Create para criar um Usuário.
- Criar o método Login que será para o usuário logar na aplicação.
- Criar o método Logout que será para o usuário sair da aplicação.
# Controllador completo de AccountController.
        public class AccountController : Controller
        {
                private UserManager<AppUser> _userManager;
                private SignInManager<AppUser> _signInManager;

                public AccountController(SignInManager<AppUser> signInManager, UserManager<AppUser> userManager)
                {
                        _userManager = userManager;
                        _signInManager = signInManager;
                }

                public IActionResult Create() => View();

                [HttpPost]
                public async Task<IActionResult> Create(User user)
                {
                        if (ModelState.IsValid)
                        {
                                AppUser newUser = new AppUser { UserName = user.UserName, Email = user.Email };
                                IdentityResult result = await _userManager.CreateAsync(newUser, user.Password);
                                if (result.Succeeded)
                                {
                                        return Redirect("/admin/products");
                                }
                                foreach (IdentityError error in result.Errors)
                                {
                                        ModelState.AddModelError("", error.Description);
                                }
                        }
                        return View(user);
                }

                public IActionResult Login(string returnUrl) => View(new LoginViewModel { ReturnUrl = returnUrl });

                [HttpPost]
                public async Task<IActionResult> Login(LoginViewModel loginVM)
                {
                        if (ModelState.IsValid)
                        {
                                Microsoft.AspNetCore.Identity.SignInResult result = await _signInManager.PasswordSignInAsync(loginVM.UserName, loginVM.Password, false, false);
                                if (result.Succeeded)
                                {
                                        return Redirect(loginVM.ReturnUrl ?? "/");
                                }
                                ModelState.AddModelError("", "Invalid username or password");
                        }
                        return View(loginVM);
                }

                public async Task<RedirectResult> Logout(string returnUrl = "/")
                {
                        await _signInManager.SignOutAsync();
                        return Redirect(returnUrl);
                }
        }

- Vamos então para a pasta Views e criar a pasta Account para fazer as Views referentes aos métodos do Controlador.
- Na pasta Views criar a pasta Account e criar a Razor View Create
# Abaixo o código da View Create
    @model User
    @{
        ViewData["Title"] = "Create";
    }
<h1>Create</h1>
<hr />
<div class="row">
        <div class="col-12">
                <form asp-action="Create">
                        <div asp-validation-summary="All" class="text-danger"></div>
                        <div class="form-group">
                                <label asp-for="UserName" class="control-label"></label>
                                <input asp-for="UserName" class="form-control" />
                                <span asp-validation-for="UserName" class="text-danger"></span>
                        </div>
                        <div class="form-group">
                                <label asp-for="Email" class="control-label"></label>
                                <input asp-for="Email" class="form-control" />
                                <span asp-validation-for="Email" class="text-danger"></span>
                        </div>
                        <div class="form-group">
                                <label asp-for="Password" class="control-label"></label>
                                <input asp-for="Password" class="form-control" />
                                <span asp-validation-for="Password" class="text-danger"></span>
                        </div>
                        <div class="form-group">
                                <button class="btn btn-primary mt-2">Register</button>
                        </div>
                </form>
        </div>
</div>

- Build no projeto!! Testar a aplicação!! http://localhost:????/account/create
- Registrar e criar um usuário.
- Verificar na tabela AspNetUsers se foi criado um registro.

- Então agora vamos criar a View de Login na mesma pasta Account em Views.
# Abaixo o código da View Login
    @model LoginViewModel
    @{
        ViewData["Title"] = "Login";
    }
<h1>Login</h1>
<hr />
<div class="row">
        <div class="col-12">
                <form asp-action="Login">
                        <div asp-validation-summary="All" class="text-danger"></div>
                        <div class="form-group">
                                <label asp-for="UserName" class="control-label"></label>
                                <input asp-for="UserName" class="form-control" />
                                <span asp-validation-for="UserName" class="text-danger"></span>
                        </div>
                        <div class="form-group">
                                <label asp-for="Password" class="control-label"></label>
                                <input asp-for="Password" class="form-control" />
                                <span asp-validation-for="Password" class="text-danger"></span>
                        </div>
                        
                        <input type="hidden" asp-for="ReturnUrl" />
                        <div class="form-group">
                                <button class="btn btn-primary mt-2">Log in</button>
                        </div>
                </form>
        </div>
</div>

- Vamos somente inserir a DataAnnotation [Authorize] no controlador ProductsController a parte de Areas

- E agora vamos começar a utilizar Identity nas Views.
- Vamos para a pasta Views - Shared no arquivo _Layout
- Abaixo do <.li de Privacy vamos inserir o Identity
# Código para inserir em _Layout
    @if (User.Identity?.IsAuthenticated ?? false)
    {
        <li class="nav-item">
            <a class="btn btn-primary" asp-area="Admin" asp-controller="Products" asp-action="Index">Products</a>
        </li>
        <li class="nav-item">
            <a class="btn btn-danger" asp-controller="Account" asp-action="Logout">Hi, @User.Identity.Name, Log out</a>
        </li>
        @if (User.IsInRole("Admin") || User.IsInRole("Editor"))
        {
            <li class="nav-item">
                <a class="btn btn-primary" asp-area="Admin" asp-controller="Products" asp-action="Index">Products</a>
            </li>
        }
    }
    else
    {
        <li class="nav-item">
            <a class="btn btn-success" asp-controller="Account" asp-action="Login">Log in</a>
        </li>
    }

- Build Solution/Project!! Testar a aplicação!!