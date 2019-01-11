# DbDeploy

## Architecture Summary 
DbUp is a database migration tool, which builds into a standard NuGet package and can be deployed using our standard Octopus deploy. It comprises of a C# console project and a TSQL script directory. 
When run the application executes the scripts in sequence and logs the execution to a SchemaVersion table. If the application is run a second time the script names are compared to the SchemaVersion table, and only scripts that are different are executed. 
 
## NuGet Packages 
Include the following NuGet packages: - 
* dbup-sqlserver 
* OctoPack 

 
## Console Application 
Replace the default console app with: 
```csharp
 public class Program  
    {  
        static int Main()  
        {  
            var connectionString = ConfigurationManager.ConnectionStrings[0].ConnectionString;  
   
            // Create Db if it doesn't exist  
            EnsureDatabase.For.SqlDatabase(connectionString);  
   
            var upgrader =  
                DeployChanges.To  
                    .SqlDatabase(connectionString)  
                    .WithScriptsEmbeddedInAssembly(Assembly.GetExecutingAssembly())  
                    .LogToConsole()  
                    .LogScriptOutput()  
                    .Build();  
   
            var result = upgrader.PerformUpgrade();  
   
            if (!result.Successful)  
            {  
                Console.ForegroundColor = ConsoleColor.Red;  
                Console.WriteLine(result.Error);  
                Console.ResetColor();  
#if DEBUG  
                Console.ReadLine();  
#endif  
                return -1;  
            }  
   
            Console.ForegroundColor = ConsoleColor.Green;  
            Console.WriteLine("Success!");  
            Console.ResetColor();  
   
#if DEBUG  
            Console.ReadLine();  
#endif  
            return 0;  
        }  
    }  
```


## Note 
* Load the connection string from an app.config file, so we can inject a deployment SQL target from Octopus. 
* EnsureDatabase.For.SqlDatabase(connectionString); 
** Crates the Database in the conneciton string if it doesn’t exist. 
* Console.ReadLine(); 
** If you are runnning a debug version in octopus then it will hang waiting for user input. 
* .LogScriptOutput() 
** Displays the output from the script in the console. 

## Scripts 
* Create a scripts folder to contain your migraiton scripts 
* Name the scripts appropiately, and try to limit each script to do one thing, it will help with debuging later. 
* Scripts are executed in alphabeticcaly name order, so prefix the name with Script001…002 etc 
* Change the property 'Build Action', on each script to Embedded Resource, so it will be included in the binary. 
* Make your sql scripts Idempotent. 
** In general you should always put guards around your scripts so that checks are made to evaluate the elements existance, or not. 
** Turn Journaling off  
*** .JournalTo(new NullJournal()) 
*** By doing this every script will execute every time. 
*** The system has no need for the schemaVersion table 
  
 
## References 
* DbUp 
** https://app.pluralsight.com/library/courses/deploying-databases-octopus/table-of-contents  
** https://dbup.github.io/  
** https://dbup.readthedocs.io/en/latest/  
** https://www.red-gate.com/simple-talk/dotnet/.net-framework/deploying-an-entity-framework-database-into-production/ 
* ElasticUp 
** https://github.com/XPCegeka/ElasticUp 
* EF SchemaCompare  
** https://github.com/JonPSmith/EfSchemaCompare 
  
  
  
  