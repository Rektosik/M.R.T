Код:
```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace HierarchicalRiskModel
{
    enum IngredientType { Positive, Negative }

    class Program
    {
        static void Main(string[] args)
        {
            Console.OutputEncoding = Encoding.UTF8;
            Console.WriteLine("=== РОЗВ'ЯЗАННЯ ЗАВДАННЯ 10 (Лабораторна робота 6) ===");
            Console.WriteLine("Побудова ієрархічної моделі багатокритеріального вибору\n");

            string[] strategies = { "x1 (Стратегія А)", "x2 (Стратегія Б)", "x3 (Стратегія В)", "x4 (Стратегія Г)" };
            string[] states = { "t1", "t2", "t3", "t4" };
            double[] probabilities = { 0.2, 0.4, 0.3, 0.1 };

            double[,] F1_Matrix = {
                { 50, 70, 40, 30 },
                { 60, 50, 50, 60 },
                { 20, 90, 30, 40 },
                { 40, 60, 55, 45 }
            };

            double[,] F2_Matrix = {
                { 10, 15, 20, 12 },
                { 12, 12, 14, 15 },
                {  8, 25, 10, 30 },
                { 15, 10, 15, 10 }
            };

            Console.WriteLine($"Кількість рішень: {strategies.Length}");
            Console.WriteLine($"Кількість станів середовища: {states.Length}");
            Console.WriteLine("--------------------------------------------------");

            Console.WriteLine("\n--- ЕТАП 1: Обробка функціоналу F1 (Прибуток, F+) ---");
            
            var vec_F1_Bayes = CalculateBayes(F1_Matrix, probabilities, IngredientType.Positive);
            var vec_F1_Wald = CalculateWald(F1_Matrix, IngredientType.Positive);
            var vec_F1_Savage = CalculateSavage(F1_Matrix, IngredientType.Positive);

            Console.WriteLine("Вектор Байєса (F1): " + VecToString(vec_F1_Bayes));
            Console.WriteLine("Вектор Вальда (F1): " + VecToString(vec_F1_Wald));
            Console.WriteLine("Вектор Севіджа (F1, перехід до F+): " + VecToString(vec_F1_Savage)); 

            var norm_F1_I1 = NormalizeVector(vec_F1_Bayes, IngredientType.Positive);
            var norm_F1_Wald = NormalizeVector(vec_F1_Wald, IngredientType.Positive);
            var norm_F1_Savage = NormalizeVector(vec_F1_Savage, IngredientType.Negative);

            double u_Wald = 0.6;
            double u_Savage = 0.4;
            var FI_1_5 = Convolve(norm_F1_Wald, u_Wald, norm_F1_Savage, u_Savage);

            Console.WriteLine("\nНормалізований I1 (Байєс): " + VecToString(norm_F1_I1));
            Console.WriteLine("Згортка I5 (0.6 Вальд + 0.4 Севідж): " + VecToString(FI_1_5));

            double u_I1 = 0.5;
            double u_I5 = 0.5;
            var FF_1 = Convolve(norm_F1_I1, u_I1, FI_1_5, u_I5);
            Console.WriteLine(">> Інтегральна оцінка FF1 (для Прибутку): " + VecToString(FF_1));

            Console.WriteLine("\n--- ЕТАП 4: Обробка функціоналу F2 (Витрати, F-) ---");
            
            var vec_F2_Bayes = CalculateBayes(F2_Matrix, probabilities, IngredientType.Negative);
            var vec_F2_Wald = CalculateWald(F2_Matrix, IngredientType.Negative);
            
            var norm_F2_I1 = NormalizeVector(vec_F2_Bayes, IngredientType.Negative);
            var norm_F2_I5 = NormalizeVector(vec_F2_Wald, IngredientType.Negative);

            var FF_2 = Convolve(norm_F2_I1, u_I1, norm_F2_I5, u_I5);
            Console.WriteLine(">> Інтегральна оцінка FF2 (для Витрат): " + VecToString(FF_2));

            Console.WriteLine("\n--- ЕТАП 6: Фінальна згортка функціоналів (ZNF) ---");
            double w_F1 = 0.7;
            double w_F2 = 0.3;

            Console.WriteLine($"Вага F1 (Прибуток): {w_F1}");
            Console.WriteLine($"Вага F2 (Витрати): {w_F2}");

            var F_Sigma = Convolve(FF_1, w_F1, FF_2, w_F2);
            Console.WriteLine("\nФІНАЛЬНИЙ РЕЙТИНГ РІШЕНЬ:");
            for (int i = 0; i < strategies.Length; i++)
            {
                Console.WriteLine($"{strategies[i]}: {F_Sigma[i]:F4}");
            }

            double maxVal = F_Sigma.Max();
            int bestIdx = Array.IndexOf(F_Sigma, maxVal);

            Console.WriteLine($" \n ОПТИМАЛЬНЕ РІШЕННЯ: {strategies[bestIdx]} з рейтингом {maxVal:F4}");

            Console.ReadKey();
        }

        static double[] CalculateBayes(double[,] matrix, double[] probs, IngredientType type)
        {
            int rows = matrix.GetLength(0);
            int cols = matrix.GetLength(1);
            double[] result = new double[rows];

            for (int i = 0; i < rows; i++)
            {
                double sum = 0;
                for (int j = 0; j < cols; j++)
                {
                    sum += matrix[i, j] * probs[j];
                }
                result[i] = sum;
            }
            return result;
        }

        static double[] CalculateWald(double[,] matrix, IngredientType type)
        {
            int rows = matrix.GetLength(0);
            int cols = matrix.GetLength(1);
            double[] result = new double[rows];

            for (int i = 0; i < rows; i++)
            {
                if (type == IngredientType.Positive)
                {
                    double min = matrix[i, 0];
                    for (int j = 1; j < cols; j++) if (matrix[i, j] < min) min = matrix[i, j];
                    result[i] = min;
                }
                else
                {
                    double max = matrix[i, 0];
                    for (int j = 1; j < cols; j++) if (matrix[i, j] > max) max = matrix[i, j];
                    result[i] = max;
                }
            }
            return result;
        }

        static double[] CalculateSavage(double[,] matrix, IngredientType type)
        {
            int rows = matrix.GetLength(0);
            int cols = matrix.GetLength(1);
            
            double[,] regretMatrix = new double[rows, cols];

            if (type == IngredientType.Positive)
            {
                for (int j = 0; j < cols; j++)
                {
                    double colMax = double.MinValue;
                    for (int i = 0; i < rows; i++) if (matrix[i, j] > colMax) colMax = matrix[i, j];
                    
                    for (int i = 0; i < rows; i++) regretMatrix[i, j] = colMax - matrix[i, j];
                }
            }
            else 
            {
                for (int j = 0; j < cols; j++)
                {
                    double colMin = double.MaxValue;
                    for (int i = 0; i < rows; i++) if (matrix[i, j] < colMin) colMin = matrix[i, j];

                    for (int i = 0; i < rows; i++) regretMatrix[i, j] = matrix[i, j] - colMin;
                }
            }

            double[] maxRegrets = new double[rows];
            for (int i = 0; i < rows; i++)
            {
                double maxR = regretMatrix[i, 0];
                for (int j = 1; j < cols; j++) if (regretMatrix[i, j] > maxR) maxR = regretMatrix[i, j];
                maxRegrets[i] = maxR;
            }

            return maxRegrets;
        }

        static double[] NormalizeVector(double[] vector, IngredientType type)
        {
            double min = vector.Min();
            double max = vector.Max();
            double denominator = max - min;
            
            if (denominator == 0) return vector.Select(x => 1.0).ToArray();

            double[] normalized = new double[vector.Length];

            for (int i = 0; i < vector.Length; i++)
            {
                if (type == IngredientType.Positive)
                {
                    normalized[i] = (vector[i] - min) / denominator;
                }
                else
                {
                    normalized[i] = (max - vector[i]) / denominator;
                }
            }
            return normalized;
        }

        static double[] Convolve(double[] v1, double w1, double[] v2, double w2)
        {
            double[] result = new double[v1.Length];
            for (int i = 0; i < v1.Length; i++)
            {
                result[i] = v1[i] * w1 + v2[i] * w2;
            }
            return result;
        }

        static string VecToString(double[] v)
        {
            return "[" + string.Join("; ", v.Select(x => x.ToString("F3"))) + "]";
        }
    }
}
```

Результат: 
<img width="701" height="552" alt="image" src="https://github.com/user-attachments/assets/3d35cf3c-6f7d-495e-ad57-da927fddcd0f" />
