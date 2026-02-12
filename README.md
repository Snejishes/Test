# Test

program 

using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Logging;

// --- 1. DAS DATENMODELL (C# Klassen) ---
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    // Die Beziehung: Ein User hat eine Liste von Posts
    public List<Post> Posts { get; set; } = new();
}

public class Post
{
    public int Id { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }
    
    // Der Fremdschlüssel (verweist auf User)
    public int UserId { get; set; }
}

// --- 2. DER ORM KONTEXT (Der Dolmetscher) ---
public class BlogContext : DbContext
{
    public DbSet<User> Users { get; set; }
    public DbSet<Post> Posts { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder options)
    {
        // Wir nutzen SQLite und aktivieren das Logging, 
        // damit wir das SQL in der Konsole SEHEN!
        options.UseSqlite("Data Source=demo.db")
               .LogTo(Console.WriteLine, LogLevel.Information);
    }
}

// --- 3. DAS HAUPTPROGRAMM (Die Demo) ---
class Program
{
    static void Main(string[] args)
    {
        using var db = new BlogContext();
        
        // Datenbank jedes Mal frisch neu erstellen (für die Demo)
        db.Database.EnsureDeleted();
        db.Database.EnsureCreated();

        Console.WriteLine("\n--- SCHRITT 1: Objekte erstellen (C#) ---");
        
        var nico = new User { Name = "Nico" };
        nico.Posts.Add(new Post { Title = "Mein erster Post", Content = "Hallo ORM!" });
        nico.Posts.Add(new Post { Title = "C# ist cool", Content = "Viel einfacher als SQL." });

        Console.WriteLine($"User '{nico.Name}' hat {nico.Posts.Count} Posts im Speicher.");

        Console.WriteLine("\n--- SCHRITT 2: Speichern (ORM Magie) ---");
        
        db.Users.Add(nico); // Wir fügen nur den User hinzu, EF Core checkt die Posts automatisch!
        db.SaveChanges();   // HIER wird das SQL generiert (siehe Konsole!)

        Console.WriteLine("\n--- SCHRITT 3: Laden (SQL Generierung) ---");
        
        // Wir löschen den Speicher, um echtes Laden zu simulieren
        db.ChangeTracker.Clear(); 

        var geladenerUser = db.Users
            .Include(u => u.Posts) // "Lade auch die Posts!"
            .First(u => u.Name == "Nico");

        Console.WriteLine($"Gelanden aus DB: {geladenerUser.Name} mit {geladenerUser.Posts.Count} Posts.");
    }
}

----
