Код:
```
using System;

class Program
{
    static void Main()
    {
        int homeRain = 8;   
        int homeSun = 3;    
        int forestRain = 1; 
        int forestSun = 10;

        Console.Write("Введiть ймовiрнiсть дощу (0..1): ");
        string input = Console.ReadLine();

        if (!double.TryParse(input, out double p))
        {
            Console.WriteLine("Помилка: потрiбно ввести число.");
            return;
        }

        if (p < 0 || p > 1)
        {
            Console.WriteLine("Помилка: ймовiрнiсть повинна бути в межах вiд 0 до 1.");
            return;
        }

        double W_home = p * homeRain + (1 - p) * homeSun;
        double W_forest = p * forestRain + (1 - p) * forestSun;

        Console.WriteLine($"\nКориснiсть вдома: {W_home:F2}");
        Console.WriteLine($"Кориснiсть у лiсi: {W_forest:F2}");

        if (W_home > W_forest)
            Console.WriteLine("Рекомендую залишитись вдома");
        else if (W_home < W_forest)
            Console.WriteLine("Рекомендую їхати в лiс");
        else
            Console.WriteLine("Рiшення: байдуже (кориснiсть однакова)");
    }
}
```
Резултат: 

1.<img width="450" height="121" alt="image" src="https://github.com/user-attachments/assets/cd824f35-797e-4ed3-8858-f670f5338700" />

2.<img width="354" height="40" alt="image" src="https://github.com/user-attachments/assets/7ab7692d-6ea6-46a9-9a79-9d538600e654" />
