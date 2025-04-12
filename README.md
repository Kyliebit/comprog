# superstick
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;

namespace PersonalBudgetTracker
{
    public class Transaction
    {
        public string Description { get; set; }
        public decimal Amount { get; set; }
        public string Type { get; set; } // "Income" or "Expense"
        public string Category { get; set; }
        public DateTime Date { get; set; }

        public Transaction(string description, decimal amount, string type, string category, DateTime date)
        {
            if (amount < 0) throw new ArgumentException("Amount must be positive.");
            if (type != "Income" && type != "Expense") throw new ArgumentException("Type must be 'Income' or 'Expense'.");

            Description = description;
            Amount = amount;
            Type = type;
            Category = category;
            Date = date;
        }

        public override string ToString()
        {
            return $"{Date.ToShortDateString()} | {Type} | {Category} | {Description} | {Amount:C}";
        }
    }

    public class BudgetTracker
    {
        private List<Transaction> transactions = new List<Transaction>();

        public void AddTransaction(Transaction t)
        {
            transactions.Add(t);
        }

        public decimal GetTotalIncome() => transactions.Where(t => t.Type == "Income").Sum(t => t.Amount);

        public decimal GetTotalExpenses() => transactions.Where(t => t.Type == "Expense").Sum(t => t.Amount);

        public decimal GetNetSavings() => GetTotalIncome() - GetTotalExpenses();

        public Dictionary<string, decimal> GetCategoryWiseSpending()
        {
            return transactions
                .Where(t => t.Type == "Expense")
                .GroupBy(t => t.Category)
                .ToDictionary(g => g.Key, g => g.Sum(t => t.Amount));
        }

        public List<Transaction> SortByDate() => transactions.OrderBy(t => t.Date).ToList();

        public List<Transaction> SortByAmount() => transactions.OrderByDescending(t => t.Amount).ToList();

        public List<Transaction> SortByCategory() => transactions.OrderBy(t => t.Category).ToList();

        public void PrintSummary()
        {
            Console.WriteLine($"\nTotal Income: {GetTotalIncome():C}");
            Console.WriteLine($"Total Expenses: {GetTotalExpenses():C}");
            Console.WriteLine($"Net Savings: {GetNetSavings():C}");

            Console.WriteLine("\nSpending by Category:");
            foreach (var kvp in GetCategoryWiseSpending())
            {
                Console.WriteLine($"{kvp.Key}: {kvp.Value:C}");
            }

            Console.WriteLine($"\nMost Spent Category: {GetMostSpentCategory()}");
        }

        public string GetMostSpentCategory()
        {
            var spending = GetCategoryWiseSpending();
            return spending.Count > 0 ? spending.OrderByDescending(kvp => kvp.Value).First().Key : "N/A";
        }

        public void DisplayTextGraph()
        {
            var categorySpending = GetCategoryWiseSpending();
            Console.WriteLine("\nText-Based Expense Graph:");
            foreach (var item in categorySpending)
            {
                Console.Write($"{item.Key}: ");
                int barLength = (int)(item.Value / 10); // scale factor
                Console.WriteLine(new string('#', barLength));
            }
        }

        public void SaveToFile(string filepath)
        {
            using (StreamWriter sw = new StreamWriter(filepath))
            {
                foreach (var t in transactions)
                {
                    sw.WriteLine($"{t.Description},{t.Amount},{t.Type},{t.Category},{t.Date}");
                }
            }
        }

        public void LoadFromFile(string filepath)
        {
            if (!File.Exists(filepath))
            {
                Console.WriteLine("File not found.");
                return;
            }

            using (StreamReader sr = new StreamReader(filepath))
            {
                string line;
                while ((line = sr.ReadLine()) != null)
                {
                    var parts = line.Split(',');
                    try
                    {
                        var t = new Transaction(
                            parts[0],
                            decimal.Parse(parts[1]),
                            parts[2],
                            parts[3],
                            DateTime.Parse(parts[4])
                        );
                        AddTransaction(t);
                    }
                    catch
                    {
                        Console.WriteLine("Invalid line skipped.");
                    }
                }
            }
        }
    }

    class Program
    {
        static void Main()
        {
            BudgetTracker tracker = new BudgetTracker();

            while (true)
            {
                Console.WriteLine("\n--- Personal Budget Tracker ---");
                Console.WriteLine("1. Add Transaction");
                Console.WriteLine("2. Show Summary");
                Console.WriteLine("3. Show Text Graph");
                Console.WriteLine("4. Save Transactions");
                Console.WriteLine("5. Load Transactions");
                Console.WriteLine("6. Exit");
                Console.Write("Choose an option: ");
                var input = Console.ReadLine();

                try
                {
                    switch (input)
                    {
                        case "1":
                            Console.Write("Description: ");
                            string desc = Console.ReadLine();

                            Console.Write("Amount: ");
                            decimal amt = decimal.Parse(Console.ReadLine());

                            Console.Write("Type (Income/Expense): ");
                            string type = Console.ReadLine();

                            Console.Write("Category: ");
                            string cat = Console.ReadLine();

                            Console.Write("Date (yyyy-mm-dd): ");
                            DateTime date = DateTime.Parse(Console.ReadLine());

                            var t = new Transaction(desc, amt, type, cat, date);
                            tracker.AddTransaction(t);
                            Console.WriteLine("Transaction added successfully.");
                            break;

                        case "2":
                            tracker.PrintSummary();
                            break;

                        case "3":
                            tracker.DisplayTextGraph();
                            break;

                        case "4":
                            tracker.SaveToFile("transactions.txt");
                            Console.WriteLine("Transactions saved to file.");
                            break;

                        case "5":
                            tracker.LoadFromFile("transactions.txt");
                            Console.WriteLine("Transactions loaded from file.");
                            break;

                        case "6":
                            Console.WriteLine("Goodbye!");
                            return;

                        default:
                            Console.WriteLine("Invalid option. Try again.");
                            break;
                    }
                }
                catch (Exception ex)
                {
                    Console.WriteLine($"Error: {ex.Message}");
                }
            }
        }
    }
}
