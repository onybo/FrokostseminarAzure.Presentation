- title : F# in Azure
- description : How to use F# to create azure website and use Azure .NET SDK from F#
- author : Olav Nybø
- theme : sky 
- transition : default

***
### Azure Web Site
- MVC og/eller Angular
- Azure DocumentDb

***

# F#

***

- Twitter; @onybo
- Mail: olav.nybo@novanet.no
- Dagtid: C# og  TypeScript hos If
- Kveldstid: F#

***

### MVC / Web API

***
    [lang=cs]
    [RoutePrefix("persons")]
    public class PersonsController : Controller
    {
      // eg.: /persons
      [Route]
      public ActionResult Index()
      {
        var persons = context.Persons
                      .Where(person => person.IsAlive())
                      .Include(p => p.Policies.Select(policy => policy.Exposures))
                      .ToList();
        return View(persons);
      }
      
      // eg.: /persons/5
      [Route("{personId}")]
      public ActionResult Show(int personId) { ... }
    }

---

    [lang=cs]
    [RoutePrefix("persons")]
    public class PersonsController : Controller
    {
      private readonly IPersonRepository _personRepository;
      
      public PersonsController(IPersonRepository personRepository)
      {
        _personRepository = personRepository;
      }
      
      // eg.: /persons
      [Route]
      public ActionResult Index()
      {
        var persons = _personsRepository.GetLivingPersonsWithPoliciesAndExposures();
        return View(persons);
      }
      
      // eg.: /persons/5
      [Route("{personId}")]
      public ActionResult Show(int personId) { ... }
    }

---

    [lang=cs]
    public interface IPersonRepository
    {
      IEnumerable<Person> GetLivingPersonsWithPoliciesAndExposures();
      void SaveChanges();
      void UpdatePerson(Person person);
      void DeletePerson(Person person);
      ...
    }

---

    [lang=cs]
    public LivingPersonsWithPoliciesAndExposuresQuery
    {
      public LivingPersonsWithPoliciesAndExposuresQuery(...)
      {
        ...
      }

      public IEnumerable<Person> GetLivingPersonsWithPoliciesAndExposures
      {
        ...
      }
    }

***

### Hvorfor F#?

    // simple types in one line
    type Person = {First:string; Last:string}
    
    // complex types in a few lines
    type Employee =
      | Worker of Person
      | Manager of Employee list
  
    // type inference
    let jdoe = {First="John";Last="Doe"}
    let worker = Worker jdoe


***
 
![null reference](images/null_reference.png) 

***

# DEMO
- C# web project med F# controllers
- F# MVC 5 / F# Web Item Templates
- ASP.NET 5 project

***

### Azure DocumentDb

- NoSql Database
- Støtter SQL som spørrespråk
- JSON dokumenter

***

### Azure DocumentDb struktur
![DocumentDb](images/DocumentDb.png)


***

### [Azure portal](https://portal.azure.com)
 
 - Create database
 - URI and Keys
 - Document Explorer
 - Query Explorer


***

### [Azure SDKs: github.com/Azure](https://github.com/Azure)
![.NET](images/Microsoft_.NET_Framework_v4.5_logo.png)
![python](images/python-logo-generic.svg)
![JS Client](images/Unofficial_JavaScript_logo_2.svg.png)
![Java](images/java-logo.jpg)
![Node](images/nodejs.png) 

***

![null reference](images/railway.png)


***

    public class DocumentsFetcher
    {
        private static string EndpointUrl = "<your endpoint URI>";
        private static string AuthorizationKey = "<your key>";

        public IEnumerable<Document> GetDocuments()
        {
            var client = new DocumentClient(new Uri(EndpointUrl), AuthorizationKey);
            if (client == null)
            {
                //log problem
                return Enumerable.Empty<Document>();
            }
            
            var database = client.CreateDatabaseAsync(
                new Database
                {
                    Id = "OlavsDemoDB"
                }).Result.Resource;

            if (database == null)
            {
               return Enumerable.Empty<Document>();
            }
            
            var documentCollection = client.CreateDocumentCollectionAsync(database.CollectionsLink,
                new DocumentCollection
                {
                    Id = "Persons"
                }).Result.Resource;
            return client.CreateDocumentQuery<Document>(documentCollection.DocumentsLink).ToList();
        }
    }

***

Railway oriented programming

![null reference](images/railway.png)

---

![null reference](images/chessie_logo.png)

    type Result<'TSuccess, 'TMessage> = 
    | Ok of 'TSuccess * 'TMessage list
    | Fail of 'TMessage list


***

# DEMO

***

# ?

***

### C# assemblies fra F#

    //module Logger

    open DocumentDbSample.Core
    open System.Diagnostics

    let trace message = 
      Trace.Write message  

    let trace2 message category =
      Trace.Write(message, category)

    let traceRecord (documentRecord:DocumentRecord) = 
      Trace.Write (sprintf "documentRecord is %A" documentRecord)

---

    [lang=cs]
    using DocumentDbSample;
    using System.Diagnostics;

    public static class Logger
    {
      public static void trace(string message)
      {
        Trace.Write(message);
      }

      public static void trace2(string message, string category)
      {
        Trace.Write(message, category);
      }

      public static void traceRecord(Core.DocumentRecord documentRecord)
      {
        FSharpFunc<Core.DocumentRecord, string> fSharpFunc =
             ExtraTopLevelOperators.PrintFormatToString<FSharpFunc<Core.DocumentRecord, string>>(
               new PrintfFormat<FSharpFunc<Core.DocumentRecord, string>, Unit, string, string, Core.DocumentRecord>(
                 "documentRecord is %A"));
        Trace.Write(fSharpFunc.Invoke(documentRecord));
      }
    }

