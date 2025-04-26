# midterm-proj-104
Izeah Ehlaine Narvas Section D


using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;

class Transaction
{
    public string Description { get; set; }
    public decimal Amount { get; set; }
    public string Type { get; set; } 
    public string Category { get; set; }
    public DateTime Date { get; set; }

    public Transaction(string description, decimal amount, string type, string category, DateTime date)
    {
        Description = description;
        Amount = amount;
        Type = type;
        Category = category;
        Date = date;
    }

    public override string ToString()
    {
        return $"{Date.ToShortDateString(),-12} | {Type,-7} | {Category,-15} | {Amount,10:N2} | {Description}";
    }
}

class BudgetTracker
{
    private List<Transaction> transactions = new List<Transaction>();
    private const int MaxTransactions = 100;

    public void AddTransaction(Transaction transaction)
    {
        if (transactions.Count >= MaxTransactions)
        {
            Console.WriteLine("Transaction limit reached (100). Cannot add more.");
            return;
        }
        transactions.Add(transaction);
    }

    public decimal GetTotalIncome() =>
        transactions.Where(t => t.Type.ToLower() == "income").Sum(t => t.Amount);

    public decimal GetTotalExpenses() =>
        transactions.Where(t => t.Type.ToLower() == "expense").Sum(t => t.Amount);

    public decimal GetNetSavings() =>
        GetTotalIncome() - GetTotalExpenses();

    public Dictionary<string, decimal> GetCategoryWiseExpenses()
    {
        return transactions
            .Where(t => t.Type.ToLower() == "expense")
            .GroupBy(t => t.Category)
            .ToDictionary(g => g.Key, g => g.Sum(t => t.Amount));
    }

    public Transaction GetHighestExpense()
    {
        return transactions
            .Where(t => t.Type.ToLower() == "expense")
            .OrderByDescending(t => t.Amount)
            .FirstOrDefault();
    }

    public List<Transaction> SortByDate() =>
        transactions.OrderBy(t => t.Date).ToList();

    public List<Transaction> SortByCategory() =>
        transactions.OrderBy(t => t.Category).ToList();

    public List<Transaction> SortByAmount() =>
        transactions.OrderByDescending(t => t.Amount).ToList();

    public void DisplayCategoryAnalytics()
    {
        var categoryExpenses = GetCategoryWiseExpenses();
        Console.WriteLine("\nCategory-wise Expense Breakdown:");
        foreach (var item in categoryExpenses)
        {
            Console.Write($"{item.Key,15}: ");
            int barLength = (int)item.Value / 10000;
            Console.WriteLine($"{new string('#', barLength)} ({item.Value:N2})");
        }
    }

    public void SaveToFile(string fileName)
    {
        using (StreamWriter writer = new StreamWriter(fileName))
        {
            foreach (var t in transactions)
            {
                writer.WriteLine($"{t.Description},{t.Amount},{t.Type},{t.Category},{t.Date}");
            }
        }
    }

    public void LoadFromFile(string fileName)
    {
        if (!File.Exists(fileName)) return;

        foreach (var line in File.ReadAllLines(fileName))
        {
            var parts = line.Split(',');
            var transaction = new Transaction(
                parts[0],
                decimal.Parse(parts[1]),
                parts[2],
                parts[3],
                DateTime.Parse(parts[4])
            );
            transactions.Add(transaction);
        }
    }
}

class Program
{
    static void Main()
    {
        BudgetTracker tracker = new BudgetTracker();
        tracker.LoadFromFile("transactions.txt");

        string[] validCategories = { "Food", "Transportation", "Savings", "Shopping", "Loans" };

        while (true)
        {
            Console.WriteLine("\n=== Personal Budget Tracker ===");
            Console.WriteLine("1. Add Transaction");
            Console.WriteLine("2. View Summary");
            Console.WriteLine("3. View Sorted Transactions");
            Console.WriteLine("4. Save and Exit");
            Console.Write("Choose option: ");
            string option = Console.ReadLine();

            try
            {
                if (option == "1")
                {
                    Console.Write("Description: ");
                    string desc = Console.ReadLine();

                    Console.Write("Amount: ");
                    decimal amount = decimal.Parse(Console.ReadLine());

                    Console.Write("Type (Income/Expense): ");
                    string type = Console.ReadLine();

                
                    string category = "";
                    while (true)
                    {
                        Console.WriteLine("Choose Category:");
                        for (int i = 0; i < validCategories.Length; i++)
                        {
                            Console.WriteLine($"{i + 1}. {validCategories[i]}");
                        }
                        Console.Write("Enter category number (1-5): ");
                        string catChoice = Console.ReadLine();

                        if (int.TryParse(catChoice, out int catIndex) &&
                            catIndex >= 1 && catIndex <= validCategories.Length)
                        {
                            category = validCategories[catIndex - 1];
                            break;
                        }
                        else
                        {
                            Console.WriteLine("Invalid category. Please try again.");
                        }
                    }

                    Console.Write("Date (yyyy-mm-dd): ");
                    DateTime date = DateTime.Parse(Console.ReadLine());

                    var transaction = new Transaction(desc, amount, type, category, date);
                    tracker.AddTransaction(transaction);
                    Console.WriteLine("Transaction added.");
                }
                else if (option == "2")
                {
                    Console.WriteLine($"\nTotal Income   : {tracker.GetTotalIncome():N2}");
                    Console.WriteLine($"Total Expenses : {tracker.GetTotalExpenses():N2}");
                    Console.WriteLine($"Net Savings    : {tracker.GetNetSavings():N2}");

                    var highest = tracker.GetHighestExpense();
                    if (highest != null)
                        Console.WriteLine($"Highest Expense: {highest}");

                    tracker.DisplayCategoryAnalytics();
                }
                else if (option == "3")
                {
                    Console.WriteLine("Sort by: 1-Date  2-Category  3-Amount");
                    var sortOption = Console.ReadLine();
                    var sorted = sortOption switch
                    {
                        "1" => tracker.SortByDate(),
                        "2" => tracker.SortByCategory(),
                        "3" => tracker.SortByAmount(),
                        _ => new List<Transaction>()
                    };

                    Console.WriteLine("\nDate         | Type    | Category        |     Amount | Description");
                    Console.WriteLine("----------------------------------------------------------------------");
                    foreach (var t in sorted)
                        Console.WriteLine(t);
                }
                else if (option == "4")
                {
                    tracker.SaveToFile("transactions.txt");
                    Console.WriteLine("Data saved. Goodbye!");
                    break;
                }
                else
                {
                    Console.WriteLine("Invalid option.");
                }
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Error: {ex.Message}");
            }
        }
    }
}
