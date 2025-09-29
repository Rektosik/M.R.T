```
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        double[] profitsOld = { 80, 50, -20 };
        double[] probsOld = { 0.2, 0.7, 0.1 };

        double[] profitsNew = { 300, 100, -100 };
        double[] probsNew = { 0.2, 0.5, 0.3 };

        var oldResults = AnalyzeModel("Стара модель", profitsOld, probsOld);
        var newResults = AnalyzeModel("Нова модель", profitsNew, probsNew);

        Console.WriteLine("\n=== ПIДСУМОК ===");
        if (oldResults.mean > newResults.mean)
            Console.WriteLine("Середнiй прибуток вищий у старої моделi.");
        else
            Console.WriteLine("Середнiй прибуток вищий у нової моделi.");

        if (oldResults.sigma < newResults.sigma)
            Console.WriteLine("Ризик менший у старої моделi.");
        else
            Console.WriteLine("Ризик менший у нової моделi.");

        if (oldResults.cv < newResults.cv)
            Console.WriteLine("Вiдносний ризик менший у старої моделi.");
        else
            Console.WriteLine("Вiдносний ризик менший у нової моделi.");

        if (oldResults.lossProb < newResults.lossProb)
            Console.WriteLine("Ймовiрнiсть збиткiв менша у старої моделi.");
        else
            Console.WriteLine("Ймовiрнiсть збиткiв менша у нової моделi.");
    }

    static (double mean, double sigma, double cv, double lossProb)
        AnalyzeModel(string name, double[] profits, double[] probs)
    {
        double mean = profits.Zip(probs, (x, p) => x * p).Sum();
        double variance = profits.Zip(probs, (x, p) => p * Math.Pow(x - mean, 2)).Sum();
        double sigma = Math.Sqrt(variance);
        double cv = sigma / mean;
        double lossProb = probs.Zip(profits, (p, x) => x < 0 ? p : 0).Sum();

        Console.WriteLine($"\n{name}:");
        Console.WriteLine($"Середнiй прибуток = {mean:F2}");
        Console.WriteLine($"Дисперсiя = {variance:F2}, sigma = {sigma:F2}");
        Console.WriteLine($"Коефiцiєнт варiацiї = {cv:F2}");
        Console.WriteLine($"Ймовiрнiсть збиткiв = {lossProb:P}");

        return (mean, sigma, cv, lossProb);
    }
}
```
Результат:

<img width="663" height="363" alt="image" src="https://github.com/user-attachments/assets/2ec7a640-fc59-4d11-8d18-a0aedc5fe033" />
