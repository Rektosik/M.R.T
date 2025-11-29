Код: 
```
using System;
using System.Linq;

namespace Lab5_Task10
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.OutputEncoding = System.Text.Encoding.UTF8;

            double p_xi1 = 0.6;
            double p_xi2 = 0.4;

            double p_theta1_given_xi1 = 0.7;
            double p_theta2_given_xi1 = 0.3;
            double p_theta1_given_xi2 = 0.4;
            double p_theta2_given_xi2 = 0.6;

            double p_theta1 = (p_theta1_given_xi1 * p_xi1) + (p_theta1_given_xi2 * p_xi2);
            double p_theta2 = (p_theta2_given_xi1 * p_xi1) + (p_theta2_given_xi2 * p_xi2);

            Console.WriteLine($"P(Theta1) = {p_theta1:F2}");
            Console.WriteLine($"P(Theta2) = {p_theta2:F2}");
            Console.WriteLine(new string('-', 30));

            int totalGoods = 1000;
            double cost = 30;
            double initialPrice = 50;

            double[] discounts = { 0.20, 0.30, 0.40 };
            int[] salesTheta1 = { 700, 800, 900 };
            int[] salesTheta2 = { 400, 500, 600 };
            string[] names = { "x1 (20%)", "x2 (30%)", "x3 (40%)" };

            double minRisk = double.MaxValue;
            int optimalIndex = -1;

            for (int i = 0; i < discounts.Length; i++)
            {
                double price = initialPrice * (1 - discounts[i]);

                double loss1 = CalculateLoss(totalGoods, salesTheta1[i], cost, price);
                double loss2 = CalculateLoss(totalGoods, salesTheta2[i], cost, price);

                double bayesCriterion = (loss1 * p_theta1) + (loss2 * p_theta2);

                Console.WriteLine($"Стратегія {names[i]}:");
                Console.WriteLine($"  Збитки при Theta1: {loss1:F2}");
                Console.WriteLine($"  Збитки при Theta2: {loss2:F2}");
                Console.WriteLine($"  Байєсівська оцінка (середні збитки): {bayesCriterion:F2}");
                Console.WriteLine();

                if (bayesCriterion < minRisk)
                {
                    minRisk = bayesCriterion;
                    optimalIndex = i;
                }
            }

            Console.WriteLine(new string('-', 30));
            Console.WriteLine($"Оптимальне рішення: {names[optimalIndex]}");
            Console.WriteLine($"Мінімальне значення функції збитків: {minRisk:F2}");
            Console.ReadKey();
        }

        static double CalculateLoss(int total, int sold, double cost, double price)
        {
            int unsold = total - sold;
            double productionCostUnsold = unsold * cost;
            double revenue = sold * price;

            return productionCostUnsold - revenue;
        }
    }
}
```
Результат: 
<img width="1094" height="463" alt="image" src="https://github.com/user-attachments/assets/2572ba69-e514-47bd-af3f-f114a7f81bd9" />
