---
title: 'NoSQL-zelfstudie: DocumentDB .NET SDK | Microsoft Docs'
description: Een NoSQL-zelfstudie waarmee u een online database en een C#-consoletoepassing maakt met de DocumentDB .NET SDK. DocumentDB is een NoSQL-database voor JSON.
keywords: nosql zelfstudie, onlinedatabase, c#-consoletoepassing
services: documentdb
documentationcenter: .net
author: AndrewHoh
manager: jhubbard
editor: monicar
ms.assetid: bf08e031-718a-4a2a-89d6-91e12ff8797d
ms.service: documentdb
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: hero-article
ms.date: 12/16/2016
ms.author: anhoh
translationtype: Human Translation
ms.sourcegitcommit: 414c41cbccb755e058fb76f10bba62bd1ffa1835
ms.openlocfilehash: d2989a3292117ada029fb26536bfffbce83f5e1b


---
# <a name="nosql-tutorial-build-a-documentdb-c-console-application"></a>NoSQL-zelfstudie: een DocumentDB C#-consoletoepassing maken
> [!div class="op_single_selector"]
> * [.NET](documentdb-get-started.md)
> * [.NET Core](documentdb-dotnetcore-get-started.md)
> * [Node.js](documentdb-nodejs-get-started.md)
> * [C++](documentdb-cpp-get-started.md)
>  
> 

Welkom bij de NoSQL-zelfstudie over de Azure DocumentDB .NET-SDK. Wanneer u deze zelfstudie hebt voltooid, beschikt u over een consoletoepassing waarmee u DocumentDB-resources kunt maken en er query's op kunt uitvoeren.

De volgende onderwerpen komen aan bod:

* Een DocumentDB-account maken en er verbinding mee maken
* Uw Visual Studio-oplossing configureren
* Een online-database maken
* Een verzameling maken
* JSON-documenten maken
* Query's uitvoeren op de verzameling
* Een document vervangen
* Een document verwijderen
* De database verwijderen

Hebt u geen tijd? Geen probleem. De volledige oplossing is beschikbaar via [GitHub](https://github.com/Azure-Samples/documentdb-dotnet-getting-started). Ga naar de sectie [De volledige NoSQL-zelfstudieoplossing ophalen](#GetSolution) voor beknopte instructies.

Gebruik daarna de stemknoppen boven of onder aan de pagina om ons feedback te geven. Als u graag rechtstreeks contact wilt opnemen, voegt u uw e-mailadres ook toe aan uw reactie.

Tijd om aan de slag te gaan.

## <a name="prerequisites"></a>Vereisten
Zorg ervoor dat u over de volgende zaken beschikt:

* Een actief Azure-account. Als u nog geen account hebt, kunt u zich aanmelden voor een [gratis account](https://azure.microsoft.com/free/). 
    * U kunt voor deze zelfstudie ook de [Azure DocumentDB-emulator](documentdb-nosql-local-emulator.md) gebruiken.
* [Visual Studio 2013/Visual Studio 2015](http://www.visualstudio.com/).
* .NET Framework 4.6

## <a name="step-1-create-a-documentdb-account"></a>Stap 1: een DocumentDB-account maken
U maakt om te beginnen een DocumentDB-account. Als u al een account hebt dat u wilt gebruiken, gaat u verder naar de stap [Uw Visual Studio-oplossing instellen](#SetupVS). Als u de DocumentDB-emulator gebruikt, volgt u de stappen in [Azure DocumentDB-emulator](documentdb-nosql-local-emulator.md) om de emulator in te stellen en meteen naar [Uw Visual Studio-oplossing instellen](#SetupVS) te gaan.

[!INCLUDE [documentdb-create-dbaccount](../../includes/documentdb-create-dbaccount.md)]

## <a name="a-idsetupvsastep-2-setup-your-visual-studio-solution"></a><a id="SetupVS"></a>Stap 2: uw Visual Studio-oplossing instellen
1. Open **Visual Studio 2015** op uw computer.
2. Selecteer in het menu **Bestand** de optie **Nieuw** en kies vervolgens **Project**.
3. Selecteer in het dialoogvenster **Nieuw project** achtereenvolgens **Sjablonen** / **Visual C#** / **Consoletoepassing**, geef een naam op voor uw op en klik vervolgens op **OK**.
   ![Schermopname van het venster Nieuw project](./media/documentdb-get-started/nosql-tutorial-new-project-2.png)
4. Klik in **Solution Explorer** met de rechtermuisknop op uw nieuwe consoletoepassing. Deze bevindt zich onder uw Visual Studio-oplossing. Klik vervolgens op **Manage NuGet Packages...**
    
    ![Schermopname van het rechtsklikmenu voor het project](./media/documentdb-get-started/nosql-tutorial-manage-nuget-pacakges.png)
5. Klik op het tabblad **Nuget** op **Bladeren** en typ **azure documentdb** in het zoekvak.
6. Zoek **Microsoft.Azure.DocumentDB** in de resultaten en klik op **Installeren**.
   De pakket-id voor de DocumentDB-clientbibliotheek is [Microsoft.Azure.DocumentDB](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB).
   ![Schermopname van het Nuget-menu voor het zoeken naar de DocumentDB Client-SDK](./media/documentdb-get-started/nosql-tutorial-manage-nuget-pacakges-2.png)

    Als u berichten ontvangt over het controleren van wijzigingen in de oplossing, klikt u op **OK**. Als u een bericht ontvangt over het accepteren van de licentie, klikt u op **Accepteren**.

Goed gedaan. De configuratie is voltooid en u kunt nu aan de slag met het schrijven van code. Op [GitHub](https://github.com/Azure-Samples/documentdb-dotnet-getting-started/blob/master/src/Program.cs) vindt u een voltooid codeproject voor deze zelfstudie.

## <a name="a-idconnectastep-3-connect-to-a-documentdb-account"></a><a id="Connect"></a>Stap 3: verbinding maken met een DocumentDB-account
Voeg eerst deze verwijzingen toe aan het begin van de C#-toepassing in het bestand Program.cs:

    using System;
    using System.Linq;
    using System.Threading.Tasks;

    // ADD THIS PART TO YOUR CODE
    using System.Net;
    using Microsoft.Azure.Documents;
    using Microsoft.Azure.Documents.Client;
    using Newtonsoft.Json;

> [!IMPORTANT]
> U moet u de bovenstaande afhankelijkheden toevoegen om deze NoSQL-zelfstudie te kunnen voltooien.
> 
> 

Voeg nu deze twee constanten en de variabele *client* toe onder de openbare klasse *Program*.

    public class Program
    {
        // ADD THIS PART TO YOUR CODE
        private const string EndpointUri = "<your endpoint URI>";
        private const string PrimaryKey = "<your key>";
        private DocumentClient client;

Ga vervolgens terug naar [Azure Portal](https://portal.azure.com) om uw URI en primaire sleutel op te halen. Uw toepassing heeft de DocumentDB-URI en de primaire sleutel nodig om te weten waarmee verbinding moet worden gemaakt en om ervoor te zorgen dat DocumentDB de verbinding van uw toepassing vertrouwt.

Navigeer in de Azure Portal naar uw DocumentDB-account en klik daarna op **Keys**.

Kopieer in de portal de URI en plak deze in `<your endpoint URI>` in het bestand program.cs. Kopieer vervolgens de PRIMAIRE SLEUTEL van de portal en plak deze in `<your key>`.

![Schermopname van Azure Portal die voor de NoSQL-zelfstudie wordt gebruikt om een C#-consoletoepassing te maken. Schermopname van Azure Portal waarin een DocumentDB-account wordt weergegeven met de hub ACTIEF gemarkeerd. Verder is de knop SLEUTELS gemarkeerd op de DocumentDB-accountblade en zijn de waarden URI, PRIMAIRE SLEUTEL en SECUNDAIRE SLEUTEL gemarkeerd op de blade Sleutels.][keys]

We starten nu de toepassing door een nieuwe instantie van de **DocumentClient** te maken.

Voeg onder de methode **Main** de nieuwe asynchrone taak met de naam **GetStartedDemo** toe, waarmee de nieuwe **DocumentClient** wordt geïnstantieerd.

    static void Main(string[] args)
    {
    }

    // ADD THIS PART TO YOUR CODE
    private async Task GetStartedDemo()
    {
        this.client = new DocumentClient(new Uri(EndpointUri), PrimaryKey);
    }

Voeg de volgende code om uw asynchrone taak met de methode **Main** uit te voeren. Uitzonderingen worden door de methode **Main** onderschept en naar de console geschreven.

    static void Main(string[] args)
    {
            // ADD THIS PART TO YOUR CODE
            try
            {
                    Program p = new Program();
                    p.GetStartedDemo().Wait();
            }
            catch (DocumentClientException de)
            {
                    Exception baseException = de.GetBaseException();
                    Console.WriteLine("{0} error occurred: {1}, Message: {2}", de.StatusCode, de.Message, baseException.Message);
            }
            catch (Exception e)
            {
                    Exception baseException = e.GetBaseException();
                    Console.WriteLine("Error: {0}, Message: {1}", e.Message, baseException.Message);
            }
            finally
            {
                    Console.WriteLine("End of demo, press any key to exit.");
                    Console.ReadKey();
            }

Druk op **F5** om uw toepassing uit te voeren. In het consolevenster wordt nu het bericht `End of demo, press any key to exit.` weergegeven met de bevestiging dat de verbinding is gemaakt.  U kunt vervolgens het consolevenster sluiten. 

Gefeliciteerd. U hebt verbinding gemaakt met een DocumentDB-account. Laten we nu eens kijken u de DocumentDB-resources kunt gebruiken.  

## <a name="step-4-create-a-database"></a>Stap 4: een database maken
Voordat u de code voor het maken van een database toevoegt, moet u een Help-methode toevoegen om naar de console te kunnen schrijven.

Kopieer de methode **WriteToConsoleAndPromptToContinue** en plak deze na de methode **GetStartedDemo**.

    // ADD THIS PART TO YOUR CODE
    private void WriteToConsoleAndPromptToContinue(string format, params object[] args)
    {
            Console.WriteLine(format, args);
            Console.WriteLine("Press any key to continue ...");
            Console.ReadKey();
    }

Uw DocumentDB-[database](documentdb-resources.md#databases) kan worden gemaakt met de methode [CreateDatabaseAsync](https://msdn.microsoft.com/library/microsoft.azure.documents.client.documentclient.createdatabaseasync.aspx) van de klasse **DocumentClient**. Een database is een logische container voor een JSON-documentopslag, gepartitioneerd in verzamelingen.

Kopieer de methode **CreateDatabaseIfNotExists** en plak deze na de methode **WriteToConsoleAndPromptToContinue**.

    // ADD THIS PART TO YOUR CODE
    private async Task CreateDatabaseIfNotExists(string databaseName)
    {
            // Check to verify a database with the id=FamilyDB does not exist
            try
            {
                    await this.client.ReadDatabaseAsync(UriFactory.CreateDatabaseUri(databaseName));
                    this.WriteToConsoleAndPromptToContinue("Found {0}", databaseName);
            }
            catch (DocumentClientException de)
            {
                    // If the database does not exist, create a new database
                    if (de.StatusCode == HttpStatusCode.NotFound)
                    {
                            await this.client.CreateDatabaseAsync(new Database { Id = databaseName });
                            this.WriteToConsoleAndPromptToContinue("Created {0}", databaseName);
                    }
                    else
                    {
                            throw;
                    }
            }
    }

Kopieer de volgende code en plak deze in de methode **GetStartedDemo** na het maken van de client. Hiermee maakt u een database met de naam *FamilyDB*.

    private async Task GetStartedDemo()
    {
        this.client = new DocumentClient(new Uri(EndpointUri), PrimaryKey);

        // ADD THIS PART TO YOUR CODE
        await this.CreateDatabaseIfNotExists("FamilyDB_oa");

Druk op **F5** om uw toepassing uit te voeren.

Gefeliciteerd. U hebt een DocumentDB-database gemaakt.  

## <a name="a-idcreatecollastep-5-create-a-collection"></a><a id="CreateColl"></a>Stap 5: een verzameling maken
> [!WARNING]
> Met **CreateDocumentCollectionAsync** maakt u een nieuwe verzameling met gereserveerde doorvoer, wat gevolgen heeft voor de kosten. Zie onze [pagina met prijzen](https://azure.microsoft.com/pricing/details/documentdb/) voor meer informatie.
> 
> 

U kunt een [verzameling](documentdb-resources.md#collections) maken met de methode [CreateDocumentCollectionAsync](https://msdn.microsoft.com/library/microsoft.azure.documents.client.documentclient.createdocumentcollectionasync.aspx) van de klasse **DocumentClient**. Een verzameling is een container van JSON-documenten en de bijbehorende JavaScript-toepassingslogica.

Kopieer en plak de methode **CreateDocumentCollectionIfNotExists** na de methode **CreateDatabaseIfNotExists**.

    // ADD THIS PART TO YOUR CODE
    private async Task CreateDocumentCollectionIfNotExists(string databaseName, string collectionName)
    {
        try
        {
            await this.client.ReadDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri(databaseName, collectionName));
            this.WriteToConsoleAndPromptToContinue("Found {0}", collectionName);
        }
        catch (DocumentClientException de)
        {
            // If the document collection does not exist, create a new collection
            if (de.StatusCode == HttpStatusCode.NotFound)
            {
                DocumentCollection collectionInfo = new DocumentCollection();
                collectionInfo.Id = collectionName;

                // Configure collections for maximum query flexibility including string range queries.
                collectionInfo.IndexingPolicy = new IndexingPolicy(new RangeIndex(DataType.String) { Precision = -1 });

                // Here we create a collection with 400 RU/s.
                await this.client.CreateDocumentCollectionAsync(
                    UriFactory.CreateDatabaseUri(databaseName),
                    collectionInfo,
                    new RequestOptions { OfferThroughput = 400 });

                this.WriteToConsoleAndPromptToContinue("Created {0}", collectionName);
            }
            else
            {
                throw;
            }
        }
    }

Kopieer de volgende code en plak deze in de methode **GetStartedDemo** na het maken van de database. Hiermee maakt u een documentverzameling met de naam *FamilyCollection_oa*.

        this.client = new DocumentClient(new Uri(EndpointUri), PrimaryKey);

        await this.CreateDatabaseIfNotExists("FamilyDB_oa");

        // ADD THIS PART TO YOUR CODE
        await this.CreateDocumentCollectionIfNotExists("FamilyDB_oa", "FamilyCollection_oa");

Druk op **F5** om uw toepassing uit te voeren.

Gefeliciteerd. U hebt een DocumentDB-documentverzameling gemaakt.  

## <a name="a-idcreatedocastep-6-create-json-documents"></a><a id="CreateDoc"></a>Stap 6: JSON-documenten maken
U kunt een [document](documentdb-resources.md#documents) maken met de methode [CreateDocumentAsync](https://msdn.microsoft.com/library/microsoft.azure.documents.client.documentclient.createdocumentasync.aspx) van de klasse **DocumentClient**. Documenten bestaan uit door gebruikers gedefinieerde (willekeurige) JSON-inhoud. U kunt nu een of meer documenten invoegen. Als u al gegevens hebt die u in de database wilt opslaan, kunt u het [hulpprogramma voor gegevensmigratie](documentdb-import-data.md) van DocumentDB gebruiken voor het importeren van gegevens in een database.

Eerst moet de klasse **Family** worden gemaakt die aangeeft welke objecten in dit voorbeeld worden opgeslagen in DocumentDB. Daarnaast moeten de subklassen **Parent**, **Child**, **Pet** en **Address** worden gemaakt die in de klasse **Family** worden gebruikt. Houd er rekening mee dat de documenten een **id**-eigenschap moeten bevatten die in JSON is geserialiseerd als **id**. Maak deze klassen door de volgende interne subklassen achter de methode **GetStartedDemo** toe te voegen.

Kopieer de klassen **Family**, **Parent**, **Child**, **Pet** en **Address** en plak deze na de methode **WriteToConsoleAndPromptToContinue**.

    private void WriteToConsoleAndPromptToContinue(string format, params object[] args)
    {
        Console.WriteLine(format, args);
        Console.WriteLine("Press any key to continue ...");
        Console.ReadKey();
    }

    // ADD THIS PART TO YOUR CODE
    public class Family
    {
        [JsonProperty(PropertyName = "id")]
        public string Id { get; set; }
        public string LastName { get; set; }
        public Parent[] Parents { get; set; }
        public Child[] Children { get; set; }
        public Address Address { get; set; }
        public bool IsRegistered { get; set; }
        public override string ToString()
        {
                return JsonConvert.SerializeObject(this);
        }
    }

    public class Parent
    {
        public string FamilyName { get; set; }
        public string FirstName { get; set; }
    }

    public class Child
    {
        public string FamilyName { get; set; }
        public string FirstName { get; set; }
        public string Gender { get; set; }
        public int Grade { get; set; }
        public Pet[] Pets { get; set; }
    }

    public class Pet
    {
        public string GivenName { get; set; }
    }

    public class Address
    {
        public string State { get; set; }
        public string County { get; set; }
        public string City { get; set; }
    }

Kopieer de methode **CreateFamilyDocumentIfNotExists** en plak deze na de methode **CreateDocumentCollectionIfNotExists**.

    // ADD THIS PART TO YOUR CODE
    private async Task CreateFamilyDocumentIfNotExists(string databaseName, string collectionName, Family family)
    {
        try
        {
            await this.client.ReadDocumentAsync(UriFactory.CreateDocumentUri(databaseName, collectionName, family.Id));
            this.WriteToConsoleAndPromptToContinue("Found {0}", family.Id);
        }
        catch (DocumentClientException de)
        {
            if (de.StatusCode == HttpStatusCode.NotFound)
            {
                await this.client.CreateDocumentAsync(UriFactory.CreateDocumentCollectionUri(databaseName, collectionName), family);
                this.WriteToConsoleAndPromptToContinue("Created Family {0}", family.Id);
            }
            else
            {
                throw;
            }
        }
    }

En voeg twee documenten in, één voor de Andersen Family en één voor de Wakefield Family.

Kopieer de volgende code en plak deze in de methode **GetStartedDemo** na het maken van de documentverzameling.

    await this.CreateDatabaseIfNotExists("FamilyDB_oa");

    await this.CreateDocumentCollectionIfNotExists("FamilyDB_oa", "FamilyCollection_oa");

    // ADD THIS PART TO YOUR CODE
    Family andersenFamily = new Family
    {
            Id = "Andersen.1",
            LastName = "Andersen",
            Parents = new Parent[]
            {
                    new Parent { FirstName = "Thomas" },
                    new Parent { FirstName = "Mary Kay" }
            },
            Children = new Child[]
            {
                    new Child
                    {
                            FirstName = "Henriette Thaulow",
                            Gender = "female",
                            Grade = 5,
                            Pets = new Pet[]
                            {
                                    new Pet { GivenName = "Fluffy" }
                            }
                    }
            },
            Address = new Address { State = "WA", County = "King", City = "Seattle" },
            IsRegistered = true
    };

    await this.CreateFamilyDocumentIfNotExists("FamilyDB_oa", "FamilyCollection_oa", andersenFamily);

    Family wakefieldFamily = new Family
    {
            Id = "Wakefield.7",
            LastName = "Wakefield",
            Parents = new Parent[]
            {
                    new Parent { FamilyName = "Wakefield", FirstName = "Robin" },
                    new Parent { FamilyName = "Miller", FirstName = "Ben" }
            },
            Children = new Child[]
            {
                    new Child
                    {
                            FamilyName = "Merriam",
                            FirstName = "Jesse",
                            Gender = "female",
                            Grade = 8,
                            Pets = new Pet[]
                            {
                                    new Pet { GivenName = "Goofy" },
                                    new Pet { GivenName = "Shadow" }
                            }
                    },
                    new Child
                    {
                            FamilyName = "Miller",
                            FirstName = "Lisa",
                            Gender = "female",
                            Grade = 1
                    }
            },
            Address = new Address { State = "NY", County = "Manhattan", City = "NY" },
            IsRegistered = false
    };

    await this.CreateFamilyDocumentIfNotExists("FamilyDB_oa", "FamilyCollection_oa", wakefieldFamily);

Druk op **F5** om uw toepassing uit te voeren.

Gefeliciteerd. U hebt twee DocumentDB-documenten gemaakt.  

![Diagram waarin u de hiërarchische relatie ziet tussen het account, de online database, de verzameling en de documenten die in de NoSQL-zelfstudie worden gebruikt om een a C#-consoletoepassing te maken](./media/documentdb-get-started/nosql-tutorial-account-database.png)

## <a name="a-idqueryastep-7-query-documentdb-resources"></a><a id="Query"></a>Stap 7: een query uitvoeren op DocumentDB-resources
DocumentDB biedt ondersteuning voor uitgebreide [query's](documentdb-sql-query.md) op de JSON-documenten die zijn opgeslagen in elke verzameling.  De volgende voorbeeldcode bevat verschillende query's, waarvoor zowel gebruik wordt gemaakt van de DocumentDB SQL-syntaxis als LINQ, die we kunnen uitvoeren voor de documenten die zijn ingevoegd tijdens de vorige stap.

Kopieer de methode **ExecuteSimpleQuery** en plak deze na de methode **CreateFamilyDocumentIfNotExists**.

    // ADD THIS PART TO YOUR CODE
    private void ExecuteSimpleQuery(string databaseName, string collectionName)
    {
        // Set some common query options
        FeedOptions queryOptions = new FeedOptions { MaxItemCount = -1 };

            // Here we find the Andersen family via its LastName
            IQueryable<Family> familyQuery = this.client.CreateDocumentQuery<Family>(
                    UriFactory.CreateDocumentCollectionUri(databaseName, collectionName), queryOptions)
                    .Where(f => f.LastName == "Andersen");

            // The query is executed synchronously here, but can also be executed asynchronously via the IDocumentQuery<T> interface
            Console.WriteLine("Running LINQ query...");
            foreach (Family family in familyQuery)
            {
                    Console.WriteLine("\tRead {0}", family);
            }

            // Now execute the same query via direct SQL
            IQueryable<Family> familyQueryInSql = this.client.CreateDocumentQuery<Family>(
                    UriFactory.CreateDocumentCollectionUri(databaseName, collectionName),
                    "SELECT * FROM Family WHERE Family.LastName = 'Andersen'",
                    queryOptions);

            Console.WriteLine("Running direct SQL query...");
            foreach (Family family in familyQueryInSql)
            {
                    Console.WriteLine("\tRead {0}", family);
            }

            Console.WriteLine("Press any key to continue ...");
            Console.ReadKey();
    }

Kopieer de volgende code en plak deze in de methode **GetStartedDemo** na het maken van het tweede document.

    await this.CreateFamilyDocumentIfNotExists("FamilyDB_oa", "FamilyCollection_oa", wakefieldFamily);

    // ADD THIS PART TO YOUR CODE
    this.ExecuteSimpleQuery("FamilyDB_oa", "FamilyCollection_oa");

Voordat u de toepassing uitvoert, klikt u in het menu **Project** op *Project name* **Properties... ** en vervolgens op **Build**. Schakel het selectievakje **Prefer 32-bit** uit, omdat routeringsgegevens van de partitie niet uit de query kunnen worden geëxtraheerd wanneer deze wordt uitgevoerd in een 32-bits proces.

Druk op **F5** om uw toepassing uit te voeren.

Gefeliciteerd. U hebt nu een query uitgevoerd op een DocumentDB-verzameling.

In het volgende diagram ziet u hoe de DocumentDB SQL-querysyntaxis wordt aangeroepen voor de verzameling u hebt gemaakt. Dezelfde logica is ook van toepassing op de LINQ-query.

![Diagram ter illustratie van het bereik en de betekenis van de query die wordt gebruikt in de NoSQL-zelfstudie om een toepassing C#-consoletoepassing te maken](./media/documentdb-get-started/nosql-tutorial-collection-documents.png)

Het trefwoord [FROM](documentdb-sql-query.md#from-clause) is optioneel in de query omdat DocumentDB-query's al zijn afgestemd op één verzameling. Daarom kan FROM Families f worden ingewisseld door FROM root r, of een andere gewenste variabelenaam. DocumentDB leidt af dat Families, root, of de variabelenaam die u hebt gekozen, standaard verwijst naar de huidige verzameling.

## <a name="a-idreplacedocumentastep-8-replace-json-document"></a><a id="ReplaceDocument"></a>Stap 8: JSON-document vervangen
DocumentDB biedt ondersteuning voor het vervangen van JSON-documenten.  

Kopieer de methode **ReplaceFamilyDocument** en plak deze na de methode **ExecuteSimpleQuery**.

    // ADD THIS PART TO YOUR CODE
    private async Task ReplaceFamilyDocument(string databaseName, string collectionName, string familyName, Family updatedFamily)
    {
        try
        {
            await this.client.ReplaceDocumentAsync(UriFactory.CreateDocumentUri(databaseName, collectionName, familyName), updatedFamily);
            this.WriteToConsoleAndPromptToContinue("Replaced Family {0}", familyName);
        }
        catch (DocumentClientException de)
        {
            throw;
        }
    }

Kopieer de volgende code en plak deze in de methode **GetStartedDemo** na de queryuitvoering, aan het einde van de methode. Nadat het document is vervangen, wordt dezelfde query opnieuw uitgevoerd om het gewijzigde document weer te geven.

    await this.CreateFamilyDocumentIfNotExists("FamilyDB_oa", "FamilyCollection_oa", wakefieldFamily);

    this.ExecuteSimpleQuery("FamilyDB_oa", "FamilyCollection_oa");

    // ADD THIS PART TO YOUR CODE
    // Update the Grade of the Andersen Family child
    andersenFamily.Children[0].Grade = 6;

    await this.ReplaceFamilyDocument("FamilyDB_oa", "FamilyCollection_oa", "Andersen.1", andersenFamily);

    this.ExecuteSimpleQuery("FamilyDB_oa", "FamilyCollection_oa");

Druk op **F5** om uw toepassing uit te voeren.

Gefeliciteerd. U hebt een DocumentDB-document vervangen.

## <a name="a-iddeletedocumentastep-9-delete-json-document"></a><a id="DeleteDocument"></a>Stap 9: JSON-document verwijderen
DocumentDB biedt ondersteuning voor het verwijderen van JSON-documenten.  

Kopieer en plak de methode **DeleteFamilyDocument** na de methode **ReplaceFamilyDocument**.

    // ADD THIS PART TO YOUR CODE
    private async Task DeleteFamilyDocument(string databaseName, string collectionName, string documentName)
    {
        try
        {
            await this.client.DeleteDocumentAsync(UriFactory.CreateDocumentUri(databaseName, collectionName, documentName));
            Console.WriteLine("Deleted Family {0}", documentName);
        }
        catch (DocumentClientException de)
        {
            throw;
        }
    }

Kopieer de volgende code en plak deze in de methode **GetStartedDemo** na de tweede queryuitvoering, aan het einde van de methode.

    await this.ReplaceFamilyDocument("FamilyDB_oa", "FamilyCollection_oa", "Andersen.1", andersenFamily);

    this.ExecuteSimpleQuery("FamilyDB_oa", "FamilyCollection_oa");

    // ADD THIS PART TO CODE
    await this.DeleteFamilyDocument("FamilyDB_oa", "FamilyCollection_oa", "Andersen.1");

Druk op **F5** om uw toepassing uit te voeren.

Gefeliciteerd. U hebt een DocumentDB-document verwijderd.

## <a name="a-iddeletedatabaseastep-10-delete-the-database"></a><a id="DeleteDatabase"></a>Stap 10: de database verwijderen
Als u de gemaakte database verwijdert, worden de database en alle onderliggende resources (verzamelingen, documenten, enz.) verwijderd.

Kopieer en plak de volgende code in de methode **GetStartedDemo** na het verwijderen van het document om de volledige database en alle onderliggende resources te verwijderen.

    this.ExecuteSimpleQuery("FamilyDB_oa", "FamilyCollection_oa");

    await this.DeleteFamilyDocument("FamilyDB_oa", "FamilyCollection_oa", "Andersen.1");

    // ADD THIS PART TO CODE
    // Clean up/delete the database
    await this.client.DeleteDatabaseAsync(UriFactory.CreateDatabaseUri("FamilyDB_oa"));

Druk op **F5** om uw toepassing uit te voeren.

Gefeliciteerd. U hebt een DocumentDB-database verwijderd.

## <a name="a-idrunastep-11-run-your-c-console-application-all-together"></a><a id="Run"></a>Stap 11: uw C#-consoletoepassing volledig uitvoeren
Druk in Visual Studio op F5 om de toepassing in de foutopsporingsmodus op te bouwen.

U ziet de uitvoer van uw GetStarted-app. De uitvoer bevat de resultaten van de query's die zijn toegevoegd en moet overeenkomen met de onderstaande voorbeeldtekst.

    Created FamilyDB_oa
    Press any key to continue ...
    Created FamilyCollection_oa
    Press any key to continue ...
    Created Family Andersen.1
    Press any key to continue ...
    Created Family Wakefield.7
    Press any key to continue ...
    Running LINQ query...
        Read {"id":"Andersen.1","LastName":"Andersen","District":"WA5","Parents":[{"FamilyName":null,"FirstName":"Thomas"},{"FamilyName":null,"FirstName":"Mary Kay"}],"Children":[{"FamilyName":null,"FirstName":"Henriette Thaulow","Gender":"female","Grade":5,"Pets":[{"GivenName":"Fluffy"}]}],"Address":{"State":"WA","County":"King","City":"Seattle"},"IsRegistered":true}
    Running direct SQL query...
        Read {"id":"Andersen.1","LastName":"Andersen","District":"WA5","Parents":[{"FamilyName":null,"FirstName":"Thomas"},{"FamilyName":null,"FirstName":"Mary Kay"}],"Children":[{"FamilyName":null,"FirstName":"Henriette Thaulow","Gender":"female","Grade":5,"Pets":[{"GivenName":"Fluffy"}]}],"Address":{"State":"WA","County":"King","City":"Seattle"},"IsRegistered":true}
    Replaced Family Andersen.1
    Press any key to continue ...
    Running LINQ query...
        Read {"id":"Andersen.1","LastName":"Andersen","District":"WA5","Parents":[{"FamilyName":null,"FirstName":"Thomas"},{"FamilyName":null,"FirstName":"Mary Kay"}],"Children":[{"FamilyName":null,"FirstName":"Henriette Thaulow","Gender":"female","Grade":6,"Pets":[{"GivenName":"Fluffy"}]}],"Address":{"State":"WA","County":"King","City":"Seattle"},"IsRegistered":true}
    Running direct SQL query...
        Read {"id":"Andersen.1","LastName":"Andersen","District":"WA5","Parents":[{"FamilyName":null,"FirstName":"Thomas"},{"FamilyName":null,"FirstName":"Mary Kay"}],"Children":[{"FamilyName":null,"FirstName":"Henriette Thaulow","Gender":"female","Grade":6,"Pets":[{"GivenName":"Fluffy"}]}],"Address":{"State":"WA","County":"King","City":"Seattle"},"IsRegistered":true}
    Deleted Family Andersen.1
    End of demo, press any key to exit.

Gefeliciteerd. U hebt de NoSQL-zelfstudie voltooid en beschikt nu over een werkende C#-consoletoepassing.

## <a name="a-idgetsolutiona-get-the-complete-nosql-tutorial-solution"></a><a id="GetSolution"></a> De volledige NoSQL-zelfstudieoplossing ophalen
Als u geen tijd hebt gehad voor het voltooien van de stappen in deze zelfstudie of als u gewoon de codevoorbeelden wilt downloaden, kunt u deze ophalen van [Github](https://github.com/Azure-Samples/documentdb-dotnet-getting-started). 

Voor het bouwen van de oplossing GetStarted hebt u het volgende nodig:

* Een actief Azure-account. Als u nog geen account hebt, kunt u zich aanmelden voor een [gratis account](https://azure.microsoft.com/free/).
* Een [DocumentDB-account][documentdb-create-account].
* De [GetStarted](https://github.com/Azure-Samples/documentdb-dotnet-getting-started)-oplossing die beschikbaar is via GitHub.

Als u de verwijzingen naar de DocumentDB .NET SDK in Visual Studio wilt herstellen, klikt u in Solution Explorer met de rechtermuisknop op de oplossing **GetStarted** en klikt u vervolgens op **Enable NuGet Package Restore** (NuGet-pakket herstellen inschakelen). Werk vervolgens in het bestand App.config de waarden bij voor EndpointUrl en AuthorizationKey zoals wordt beschreven in [Verbinding maken met een DocumentDB-account](#Connect).

Voordat u de toepassing uitvoert, klikt u in het menu **Project** op *Project name* **Properties... ** en vervolgens op **Build**. Schakel het selectievakje **Prefer 32-bit** uit.

Dat is alles, bouw nu de oplossing. Succes!


## <a name="next-steps"></a>Volgende stappen
* Wilt u een complexere ASP.NET MVC NoSQL-zelfstudie? Zie [ Een ASP.NET MVC-webtoepassing bouwen met DocumentDB](documentdb-dotnet-application.md).
* Wilt u de schaal en prestaties testen met DocumentDB? Zie [Prestaties en schaal testen met Azure DocumentDB](documentdb-performance-testing.md).
* Informatie over [het bewaken van een DocumentDB-account](documentdb-monitor-accounts.md).
* Voer query's uit op onze voorbeeldgegevensset in de [Queryspeelplaats](https://www.documentdb.com/sql/demo).
* Meer informatie over het programmeermodel vindt u in de sectie Ontwikkelen van de pagina [DocumentDB-documentatie](https://azure.microsoft.com/documentation/services/documentdb/).

[documentdb-create-account]: documentdb-create-account.md
[documentdb-manage]: documentdb-manage.md
[keys]: media/documentdb-get-started/nosql-tutorial-keys.png



<!--HONumber=Dec16_HO3-->


